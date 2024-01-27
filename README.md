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
  if typeof(Ball) == "Instance" and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) then
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
  local ballHitPart = false

  if not VerifyBall(Ball) then
    return
  end

  local OldPosition = Ball.Position
  local OldTick = tick()

  local function HandleBallUpdate()
    local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
    local Velocity = (OldPosition - Ball.Position).Magnitude

    local newSize = Vector3.new(5, 5, 5) + Vector3.new(Velocity / 2, Velocity / 2, Velocity / 2)

    local scaleTweenInfo = TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut)

    if ballSpawned then
      local scaleTween = TweenService:Create(BallPart, scaleTweenInfo, {Size = newSize})
      scaleTween:Play()
    else
      BallPart.Size = newSize
      ballSpawned = true
    end

    if (Distance / Velocity) <= 10 then
      HandleBallHit()
    end

    OldPosition = Ball.Position
  end

  Ball:GetPropertyChangedSignal("Position"):Connect(HandleBallUpdate)
end)

BallPart.Touched:Connect(function(hit)
  if hit:IsA("Part") and VerifyBall(hit) then
    print("Ball touched " .. hit.Name)
  end
end)

game:GetService("RunService").Heartbeat:Connect(function()
  if Player.Character then
    BallPart.Position = Player.Character.HumanoidRootPart.Position

    if IsTarget() then
     
    end
  end
end)
