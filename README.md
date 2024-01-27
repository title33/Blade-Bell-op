local StatsService = game:GetService("Stats")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local Player = Players.LocalPlayer

local Saved = {
  LastTick = tick(),
  LastBallPosition = nil,
  AttemptedParry = false,
}

local function GetBall()
  local RealBall, OtherBall = nil, nil
  for _, Object in pairs(workspace.Balls:GetChildren()) do
    if Object:GetAttribute("realBall") == true then
      RealBall = Object
    else
      OtherBall = Object
    end
  end
  return RealBall, OtherBall
end

local function AttemptParry(OtherBall)
  ReplicatedStorage.Remotes.ParryAttempt:FireServer(1.5, OtherBall.CFrame, (function()
    local Results = {}
    for _, Character in pairs(workspace.Alive:GetChildren()) do
      Results[Character.Name] = Character.HumanoidRootPart.Position
    end
    return Results
  end)(), { math.random(100, 999), math.random(100, 999) })
end

while task.wait() do
  local RealBall, OtherBall = GetBall()
  if RealBall ~= nil and OtherBall ~= nil then
    if Saved.LastBallPosition ~= nil then
      if RealBall:GetAttribute("target") == Player.Name then
        local DeltaT = tick() - Saved.LastTick
        local VelocityX = (OtherBall.Position.X - Saved.LastBallPosition.X) / DeltaT
        local VelocityY = (OtherBall.Position.Y - Saved.LastBallPosition.Y) / DeltaT
        local VelocityZ = (OtherBall.Position.Z - Saved.LastBallPosition.Z) / DeltaT
        local VelocityMagnitude = math.sqrt(VelocityX^2 + VelocityY^2 + VelocityZ^2)

        local ServerPing = StatsService.Network.ServerStatsItem["Data Ping"]:GetValue()
        local DistanceToPlayer = (Player.Character.HumanoidRootPart.Position - OtherBall.Position).Magnitude
        local EstimatedTimeToReachPlayer = (ServerPing / VelocityMagnitude) / (ServerPing / DistanceToPlayer)
        local TimeToParry = 0.2 * (VelocityMagnitude / DistanceToPlayer)

        print(EstimatedTimeToReachPlayer, "<=", TimeToParry)

        if tostring(EstimatedTimeToReachPlayer) ~= "inf" and TimeToParry < 10 then
          if EstimatedTimeToReachPlayer <= TimeToParry then
            if not Saved.AttemptedParry then
              warn("--firing")
              AttemptParry(OtherBall)
              Saved.AttemptedParry = true
            else
              warn("--attempted")
            end
          else
            Saved.AttemptedParry = false
          end
        end
      else
        Saved.AttemptedParry = false
      end
    end
    Saved.LastBallPosition = OtherBall.Position
  end
  Saved.LastTick = tick()
end
