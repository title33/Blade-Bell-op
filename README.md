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

local function UpdateIndicatorSize(BallSpeed)
    local SizeMultiplier = 1.5
    local NewSize = Vector3.new(1, 1, 1) * BallSpeed * SizeMultiplier
    Ball.Size = NewSize
end

-- The actual code

Balls.ChildAdded:Connect(function(Ball)
    if not VerifyBall(Ball) then
        return
    end

    print("Ball Spawned: " .. Ball.Name)

    local OldPosition = Ball.Position
    local OldTick = tick()

    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if IsTarget() then
            local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
            local Velocity = (OldPosition - Ball.Position).Magnitude

            print("Distance: " .. Distance .. "\nVelocity: " .. Velocity .. "\nTime: " .. Distance / Velocity)

            if (Distance / Velocity) <= 10 then
                Parry()
            end

            UpdateIndicatorSize(Velocity)
        end

        if (tick() - OldTick >= 1/60) then
            OldTick = tick()
            OldPosition = Ball.Position
        end
    end)
end)

local Ball = Instance.new("Part")
Ball.Size = Vector3.new(20, 20, 20)
Ball.Shape = Enum.PartType.Ball
Ball.Material = Enum.Material.ForceField
Ball.CanQuery = false
Ball.CanTouch = false
Ball.CanCollide = false
Ball.CastShadow = false
Ball.Color = Color3.fromRGB(255, 255, 255)
Ball.Parent = workspace

workspace.DescendantAdded:Connect(function(descendant)
    if descendant:IsA("Part") and descendant.Name ~= "Ball" then
        descendant.Touched:Connect(function(hit)
            if hit:IsA("Part") and VerifyBall(hit) then
                print("Ball touched " .. hit.Name)
            end
        end)
    end
end)

game:GetService("RunService").Heartbeat:Connect(function()
    if Player.Character then
        Ball.Position = Player.Character.HumanoidRootPart.Position
    end
end)
