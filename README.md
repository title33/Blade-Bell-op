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

            if (BallData.Distance / BallData.Velocity) <= 9 then
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

local player = game:GetService("Players").LocalPlayer
local workspace = game:GetService("Workspace")

local part = Instance.new("Part")
part.Size = Vector3.new(4, 4, 4)
part.Shape = Enum.PartType.Ball
part.BrickColor = BrickColor.new("Really black")
part.Transparency = 0.5
part.CanCollide = false
part.Parent = workspace

function CalculateParryDistance(ballSpeed)
    local factor = 1.4
    return ballSpeed / factor
end

function CheckBall()
    for i, v in pairs(workspace.Balls:GetChildren()) do
        if v:GetAttribute("target") == game.Players.LocalPlayer.Name then
            return {true, v, v:GetAttribute("target"), v.Velocity.Magnitude}
        end
    end
    return {false}
end

game:GetService("RunService").Heartbeat:Connect(function()
    if player.Character then
        part.Position = player.Character.HumanoidRootPart.Position
    end
    local ballData = CheckBall()
    if ballData[1] then
        local ballSpeed = ballData[4]
        local requiredDistance = CalculateParryDistance(ballSpeed)
        
        local ballPosition = ballData[2].Position
        if (ballPosition - part.Position).Magnitude <= requiredDistance then
            game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
        end
    end

    if ballData[1] then
        local ballSpeed = ballData[4]
        local newSize = Vector3.new(4, 4, 4) + Vector3.new(ballSpeed / 4, ballSpeed / 4, ballSpeed / 4)
        part.Size = newSize
    else

        part.Size = Vector3.new(4, 4, 4)
    end

    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Xylo Hub",
        Text = "Auto Parry Blade Ball 99%",
        Duration = 5
    })
end)
