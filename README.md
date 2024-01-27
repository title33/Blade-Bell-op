local player = game:GetService("Players").LocalPlayer
local workspace = game:GetService("Workspace")

local Ball = Instance.new("Part")
Ball.Size = Vector3.new(20, 20, 20)
Ball.Shape = Enum.PartType.Ball
Ball.Material = Enum.Material.ForceField
Ball.CanQuery = false
Ball.CanTouch = false
Ball.CanCollide = false
Ball.CastShadow = false
Ball.Color = Color3.fromRGB(255, 255, 255)
Ball.Parent = workspace

game:GetService("RunService").Heartbeat:Connect(function()
    if player.Character then
        Ball.Position = player.Character.HumanoidRootPart.Position
    end
end) 
