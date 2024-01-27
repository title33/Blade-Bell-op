local Debug = false
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
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

local BallPart = Instance.new("Part")
BallPart.Size = Vector3.new(5, 5, 5)
BallPart.Shape = Enum.PartType.Ball
BallPart.Material = Enum.Material.ForceField
BallPart.CanQuery = false
BallPart.CanTouch = false
BallPart.CanCollide = false
BallPart.CastShadow = false
BallPart.Color = Color3.fromRGB(255, 255, 255)
BallPart.Parent = workspace

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

            print("Distance: " .. Distance .. ", Velocity: " .. Velocity)

            -- Calculate new size based on ball speed
            local newSize = Vector3.new(5, 5, 5) + Vector3.new(Velocity / 4, Velocity / 4, Velocity / 4)

            -- Use TweenService for smoother scaling
            local scaleTweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut)
            local scaleTween = TweenService:Create(BallPart, scaleTweenInfo, {Size = newSize})
            scaleTween:Play()

            if (Distance / Velocity) <= 10 then
                Parry()
            end
        end

        if (tick() - OldTick >= 1/60) then
            OldTick = tick()
            OldPosition = Ball.Position
        end
    end)
end)

workspace.DescendantAdded:Connect(function(descendant)
    if descendant:IsA("Part") and descendant.Name ~= "Ball" then
        descendant.Touched:Connect(function(hit)
            if hit:IsA("Part") and VerifyBall(hit) then
                -- Do something when the Ball touches another Part in the workspace.
                print("Ball touched " .. hit.Name)
            end
        end)
    end
end)

game:GetService("RunService").Heartbeat:Connect(function()
    if Player.Character then
        BallPart.Position = Player.Character.HumanoidRootPart.Position
    end
end)
