local BallPart = Instance.new("Part")
BallPart.Size = Vector3.new(45, 45, 45)
BallPart.Shape = Enum.PartType.Ball
BallPart.Material = Enum.Material.ForceField
BallPart.CanQuery = false
BallPart.CanTouch = false  -- Keep this false for visual purposes
BallPart.CanCollide = false
BallPart.CastShadow = false
BallPart.Color = Color3.fromRGB(255, 255, 255)
BallPart.Parent = workspace

local player = game.Players.LocalPlayer or game.Players.PlayerAdded:Wait()

game:GetService("RunService").Heartbeat:Connect(function()
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        BallPart.Position = player.Character.HumanoidRootPart.Position
    end
end)

function CheckBall()
    for i, v in pairs(workspace.Balls:GetChildren()) do
        if v:GetAttribute("target") ~= "" then
            return {true, v, v:GetAttribute("target"), v.Velocity.Magnitude}
        end
    end
    return {false}
end

BallPart.Touched:Connect(function(hit)
    if hit:IsA("Part") and hit.Parent == workspace.Balls then  -- Validate ball collision
        local ballData = CheckBall()
        if ballData[1] then
            local distance = player:DistanceFromCharacter(ballData[2].Position)
            local requiredDistance = ballData[4] / 1.5
            if distance <= requiredDistance then
                game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
            end
        end
    end
end)
