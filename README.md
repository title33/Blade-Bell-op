local Debug = false
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local workspace = game:GetService("Workspace")

local Player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 9e9)
local Balls = workspace:WaitForChild("Balls", 9e9)

loadstring(game:GetObjects("rbxassetid://15900013841")[1].Source)()

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

local ballSpawned = false

Balls.ChildAdded:Connect(function(Ball)
    if not VerifyBall(Ball) then
        return
    end

    local OldPosition = Ball.Position
    local OldTick = tick()

    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if IsTarget() then
            local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
            local Velocity = (OldPosition - Ball.Position).Magnitude

            local newSize = Vector3.new(5, 5, 5) + Vector3.new(Velocity / 5, Velocity / 5, Velocity / 5)

            local scaleTweenInfo = TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut)

            if ballSpawned then
                local scaleTween = TweenService:Create(BallPart, scaleTweenInfo, {Size = newSize})
                scaleTween:Play()
            else
                BallPart.Size = newSize
                ballSpawned = true
            end

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
