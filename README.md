local player = game.Players.LocalPlayer
local character = player.Character

local circleShape = newCircle(20, 0, 0)
local circleColor = newColor3(1, 1, 1)

local circle = Instance.new("Circle")
circle.Shape = circleShape
circle.Color = circleColor

circle.Parent = character

circle.Anchored = true
circle.Position = character.Torso.Position

circle.Changed:Connect(function()
  circle.Position = character.Torso.Position
end)

while true do
  wait()
end
