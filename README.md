local Debug = false
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local workspace = game:GetService("Workspace")

local Player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 9e9)
local Balls = workspace:WaitForChild("Balls", 9e9)

-- Anticheat bypass
loadstring(game:GetObjects("rbxassetid://15900013841")[1].Source)()

-- Functions

local function print(...) -- Debug print.
    if Debug then
        warn(...)
    end
end

local function VerifyBall(Ball) -- Returns nil if the ball isn't a valid projectile; true if it's the right ball.
    if typeof(Ball) == "Instance" and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) and Ball:GetAttribute("realBall") == true then
        return true
    end
end

local function IsTarget() -- Returns true if we are the current target.
    return (Player.Character and Player.Character:FindFirstChild("Highlight"))
end

local function Parry() -- Parries.
    Remotes:WaitForChild("ParryButtonPress"):Fire()
end

-- The actual code

Balls.ChildAdded:Connect(function(Ball)
    if not VerifyBall(Ball) then
        return
    end

    print("Ball Spawned: " .. Ball.Name)

    local BallPart = Instance.new("Part")
    BallPart.Size = Vector3.new(20, 20, 20)
    BallPart.Shape = Enum.PartType.Ball
    BallPart.Material = Enum.Material.ForceField
    BallPart.CanQuery = false
    BallPart.CanTouch = false
    BallPart.CanCollide = false
    BallPart.CastShadow = false
    BallPart.Color = Color3.fromRGB(255, 255, 255)
    BallPart.Parent = workspace

    local OldPosition = Ball.Position
    local OldTick = tick()

    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if IsTarget() then -- No need to do the math if we're not being attacked.
            local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
            local Velocity = (OldPosition - Ball.Position).Magnitude -- Fix for .Velocity not working. Yes, I got the lowest possible grade in accuplacer math.

            print("Distance: " .. Distance .. "\nVelocity: " .. Velocity .. "\nTime: " .. Distance / Velocity)

            if (Distance / Velocity) <= 10 then -- Sorry for the magic number. This just works. No, you don't get a slider for this because it's 2 am.
                Parry()
            end

            -- Adjust the size of the BallPart based on Velocity
            local newSize = Vector3.new(20, 20, 20) * Velocity
            BallPart.Size = newSize
        end

        if (tick() - OldTick >= 1/60) then -- Don't want it to update too quickly because my velocity implementation is aids. Yes, I tried Ball.Velocity. No, it didn't work.
            OldTick = tick()
            OldPosition = Ball.Position
        end
    end)
end)

game:GetService("RunService").Heartbeat:Connect(function()
    if Player.Character then
        -- Update the position of the BallPart to follow the player
        BallPart.Position = Player.Character.HumanoidRootPart.Position
    end
end)
