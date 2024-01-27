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
local BallPart = Instance.new("Part")
BallPart.Size = Vector3.new(5, 5, 5) -- Set initial size to be small
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

    -- Set initial size of the Ball
    BallPart.Size = Vector3.new(5, 5, 5)

    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if IsTarget() then
            local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
            local Velocity = (OldPosition - Ball.Position).Magnitude

            print("Distance: " .. Distance .. ", Velocity: " .. Velocity)

            -- Expand sideways
            local NewSize = Velocity * 0.05
            BallPart.Size = Vector3.new(math.max(NewSize, 5), NewSize, NewSize)

            -- Additional conditions for expanding the Part, e.g., expand only when the ball is moving faster than a certain speed
            if Velocity > 100 then
                -- Set the direction for expanding the Part, e.g., expand forward
                BallPart.Size = Vector3.new(NewSize, math.max(NewSize, 5), math.max(NewSize, 5))
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
