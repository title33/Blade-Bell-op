local Debug = false 
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local Player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local Balls = workspace:WaitForChild("Balls")

-- Anticheat bypass (assuming it's legitimate and necessary)
loadstring(game:GetObjects("rbxassetid://15900013841")[1].Source)()

-- Functions
local function print(...)
    if Debug then
        warn(...)
    end
end

local function VerifyBall(ball)
    if typeof(ball) == "Instance" and ball:IsA("BasePart") and ball:IsDescendantOf(Balls) and ball:GetAttribute("realBall") == true then
        return true
    end
end

local function IsTarget()
    return Player.Character and Player.Character:FindFirstChild("Highlight")
end

local function Parry()
    Remotes:WaitForChild("ParryButtonPress"):FireServer() 
end

-- Main logic
Balls.ChildAdded:Connect(function(ball)
    if not VerifyBall(ball) then
        return
    end

    print(`Ball Spawned: {ball}`)

    local oldPosition = ball.Position
    local lastUpdate = tick()

    RunService.Heartbeat:Connect(function()
        if IsTarget() then
            local distance = (ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
            local velocity = (ball.Position - oldPosition).Magnitude / (tick() - lastUpdate)  -- คำนวณความเร็วที่แม่นยำ

            print(`Distance: {distance}\nVelocity: {velocity}\nTime: {distance / velocity}`)

            
            local adjustedTime = tick() - Player.NetworkPing / 2
            local predictedPosition = ball.Position + ball.Velocity * adjustedTime

            if (player.Character.Position - predictedPosition).Magnitude <= 2 then 
                Parry()
            end
        end

        oldPosition = ball.Position
        lastUpdate = tick()
    end)
end)
