local workspace = game:GetService("Workspace")
local players = game:GetService("Players")
local runService = game:GetService("RunService")
local vim = game:GetService("VirtualInputManager")

local ballFolder = workspace.Balls
local indicatorPart = Instance.new("Part")
indicatorPart.Anchored = true
indicatorPart.CanCollide = false
indicatorPart.Transparency = 0.5
indicatorPart.BrickColor = BrickColor.new("Bright red")
indicatorPart.Parent = workspace

local lastBallPressed = nil
local isKeyPressed = false

local function calculatePredictionTime(ball, player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local rootPart = player.Character.HumanoidRootPart
        local relativePosition = ball.Position - rootPart.Position
        local velocity = ball.Velocity + rootPart.Velocity 
        local a = ball.Size.magnitude / 2
        local b = relativePosition.magnitude
        local c = math.sqrt(a * a + b * b)
        local timeToCollision = (c - a) / velocity.magnitude
        return timeToCollision, velocity.magnitude
    end
    return math.huge, 0
end

local function updateIndicatorSize(ball, speed)
    local sizeMultiplier = 1.5 
    local newSize = Vector3.new(1, 1, 1) * speed * sizeMultiplier
    indicatorPart.Size = newSize
end

local function updateIndicatorPosition(ball)
    indicatorPart.Position = ball.Position
end

local function isPlayerTarget(ball, player)
    local realBallAttribute = ball:GetAttribute("realBall")
    local target = ball:GetAttribute("target")
    return realBallAttribute == true and target == player.Name
end

local function checkProximityToPlayer(ball, player)
    local predictionTime, ballSpeed = calculatePredictionTime(ball, player)
    
    local ballSpeedThreshold = math.max(0.4, 0.6 - ballSpeed * 0.01)

    if predictionTime <= ballSpeedThreshold and isPlayerTarget(ball, player) and not isKeyPressed then
        vim:SendKeyEvent(true, Enum.KeyCode.F, false, nil)
        wait(0.005)
        vim:SendKeyEvent(false, Enum.KeyCode.F, false, nil)
        lastBallPressed = ball
        isKeyPressed = true
    elseif lastBallPressed == ball and (predictionTime > ballSpeedThreshold or not isPlayerTarget(ball, player)) then
        isKeyPressed = false
    end

  
    if predictionTime ~= math.huge then
        updateIndicatorSize(ball, ballSpeed)
        updateIndicatorPosition(ball)
    end
end

local function checkPlayerCollision()
    local player = players.LocalPlayer
    if player and player.Character then
        local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
        if rootPart then
            local distance = (indicatorPart.Position - rootPart.Position).Magnitude
            local collisionDistance = (indicatorPart.Size.X / 2 + rootPart.Size.X / 2)
            
            if distance <= collisionDistance and isPlayerTarget(lastBallPressed, player) then
                -- Collision occurred and player is the target, send key event
                vim:SendKeyEvent(true, Enum.KeyCode.F, false, nil)
                wait(0.005)
                vim:SendKeyEvent(false, Enum.KeyCode.F, false, nil)
            end
        end
    end
end

local function checkBallsProximity()
    local player = players.LocalPlayer
    if player then
        for _, ball in pairs(ballFolder:GetChildren()) do
            checkProximityToPlayer(ball, player)
        end
    end
end

runService.Heartbeat:Connect(checkBallsProximity)
runService.Heartbeat:Connect(checkPlayerCollision)

print("Script ran without errors")

