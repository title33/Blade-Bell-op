local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")

local circleRadius = 20
local circleColor = Color3.new(1, 1, 1)

local circle = Instance.new("Circle")
circle.Radius = circleRadius
circle.Color = circleColor
circle.Parent = workspace

runService.RenderStepped:Connect(function()
    if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local rootPart = player.Character.HumanoidRootPart
        circle.Position = Vector2.new(rootPart.Position.X, rootPart.Position.Z)
    end
end)
