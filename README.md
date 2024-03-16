local B = Instance.new("Part")
B.Size = Vector3.new(5, 5, 5)
B.Shape = Enum.PartType.Ball
B.Material = Enum.Material.ForceField
B.CanQuery = false
B.CanTouch = false
B.CanCollide = false
B.CastShadow = false
B.Color = Color3.fromRGB(255, 255, 255)
B.Parent = workspace

-- เลือกผู้เล่นที่ต้องการให้วัตถุติดตาม
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- สร้างการอัปเดตตำแหน่งของวัตถุ Ball ตามตำแหน่งของผู้เล่น
game:GetService("RunService").Heartbeat:Connect(function()
    B.Position = humanoid.RootPart.Position
end)

local Debug = false -- Set this to true if you want my debug output.
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local Player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 9e9) -- A second argument in waitforchild what could it mean?
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
    
    local OldPosition = Ball.Position
    local OldTick = tick()
    
    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if not IsTarget() then
            local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
            local Velocity = (OldPosition - Ball.Position).Magnitude
            
            print("Distance: " .. Distance)
            print("Velocity: " .. Velocity)
            print("Time: " .. Distance / Velocity)
            
            -- Adjust B's size based on the velocity of the ball
            local newSize = Vector3.new(Velocity, Velocity, Velocity)
            B.Size = newSize
            
            -- Update the position of B to follow the humanoid
            B.Position = humanoid.RootPart.Position
        end
        
        if (tick() - OldTick >= 1/60) then
            OldTick = tick()
            OldPosition = Ball.Position
        end
    end)
end)
