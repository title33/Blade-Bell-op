local Debug = false -- Set this to true if you want my debug output.
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local Player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Remotes = ReplicatedStorage:WaitForChild("Remotes", math.huge) -- Set the second argument to a very large number
local Balls = workspace:WaitForChild("Balls", math.huge) -- Set the second argument to a very large number

-- Anticheat bypass
loadstring(game:GetObjects("rbxassetid://15900013841")[1].Source)()

-- Functions

local function print(...) -- Debug print.
    if Debug then
        warn(...)
    end
end

local function VerifyBall(Ball) -- Returns nil if the ball isn't a valid projectile; true if it's the right ball.
    return typeof(Ball) == "Instance" and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) and Ball:GetAttribute("realBall") == true
end

local function IsTarget() -- Returns true if we are the current target.
    return Player.Character and Player.Character:FindFirstChild("Highlight") ~= nil
end

local function Parry() -- Parries.
    Remotes:WaitForChild("ParryButtonPress"):Fire()
end

local function CalculateProjectileTime(initialPosition, targetPosition, initialVelocity)
    local distance = (targetPosition - initialPosition).Magnitude
    return distance / initialVelocity.Magnitude
end

local function CalculateDistance(projectilePosition, objectPosition)
    return (projectilePosition - objectPosition).Magnitude
end

local function onBallAdded(Ball)
    if not VerifyBall(Ball) then
        return
    end
    
    print("Ball Spawned:", Ball)
    
    local lastPosition = Ball.Position
    local lastTick = tick()
    
    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if IsTarget() then -- No need to do the math if we're not being attacked.
            local distance = CalculateDistance(Ball.Position, workspace.CurrentCamera.Focus.Position)
            local velocity = (lastPosition - Ball.Position).Magnitude -- Fix for .Velocity not working. Yes I got the lowest possible grade in accuplacer math.
            
            print("Distance:", distance, "Velocity:", velocity, "Time:", CalculateProjectileTime(Ball.Position, workspace.CurrentCamera.Focus.Position, Ball.Velocity))
        
            if (distance / velocity) <= 10 then -- Sorry for the magic number. This just works. No, you don't get a slider for this because it's 2am.
                Parry()
            end
        end
        
        if (tick() - lastTick >= 1/60) then -- Don't want it to update too quickly because my velocity implementation is aids. Yes, I tried Ball.Velocity. No, it didn't work.
            lastTick = tick()
            lastPosition = Ball.Position
        end
    end)
end

Balls.ChildAdded:Connect(onBallAdded)

