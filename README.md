game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "Xylo Hub",
    Text = "Auto Parry Blade Ball 99%",
    Duration = 5
})

local Debug = false
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local Player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 9e9)
local Balls = workspace:WaitForChild("Balls", 9e9)

local function print(...)
    if Debug then
        warn(...)
    end
end

local function VerifyBall(Ball)
    if typeof(Ball) == "Instance" and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) and Ball:GetAttribute("realBall") == true then
        return true
    end
end

local function IsTarget()
    return (Player.Character and Player.Character:FindFirstChild("Highlight"))
end

local function Parry()
    Remotes:WaitForChild("ParryButtonPress"):Fire()
end

local function TrackBallData(Ball)
    local BallData = {
        Distance = 0,
        Velocity = 0
    }

    local OldPosition = Ball.Position
    local OldTick = tick()

    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if IsTarget() then
            local CurrentPosition = Ball.Position
            BallData.Distance = (CurrentPosition - workspace.CurrentCamera.Focus.Position).Magnitude
            BallData.Velocity = (OldPosition - CurrentPosition).Magnitude

            print("Distance:", BallData.Distance)
            print("Velocity:", BallData.Velocity)
            print("Time:", BallData.Distance / BallData.Velocity)

            if (BallData.Distance / BallData.Velocity) <= 10 then
                Parry()
            end
        end

        if (tick() - OldTick >= 1/60) then
            OldTick = tick()
            OldPosition = Ball.Position
        end
    end)

    return BallData
end

local TrackedBalls = {}

Balls.ChildAdded:Connect(function(Ball)
    if not VerifyBall(Ball) then
        return
    end

    print("Ball Spawned:", Ball)
    local BallData = TrackBallData(Ball)
    table.insert(TrackedBalls, BallData)
end)


