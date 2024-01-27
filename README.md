local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")

local ballRadius = 2.5
local ballColor = Color3.new(1, 1, 1)

local ball = Instance.new("Part")
ball.Size = Vector3.new(ballRadius * 2, ballRadius * 2, ballRadius * 2)
ball.Shape = Enum.PartType.Ball
ball.Material = Enum.Material.ForceField
ball.CanQuery = false
ball.CanTouch = false
ball.CanCollide = false
ball.CastShadow = false
ball.Color = ballColor
ball.Parent = workspace

runService.RenderStepped:Connect(function()
    if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local rootPart = player.Character.HumanoidRootPart
        ball.Position = rootPart.Position
    end
end)
