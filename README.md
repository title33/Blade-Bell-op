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

local player = game.Players.LocalPlayer or game.Players.PlayerAdded:Wait()

-- ติดตาม HumanoidRootPart ของผู้เล่น
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

local lastBallPart = nil  -- เพิ่มตัวแปรเพื่อตรวจสอบว่า BallPart ได้ถูกชนกันหรือไม่

while true do
    wait(0.002)
    local ballData = CheckBall()
    if ballData[1] and ballData[3] == player.Name then
        local velocity = ballData[4]
        BallPart.Size = Vector3.new(velocity, velocity, velocity)
        
        if BallPart ~= lastBallPart then
            lastBallPart = BallPart
            game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
        end
    else
        lastBallPart = nil  -- รีเซ็ตตัวแปรเมื่อไม่มีบอลชน
    end
    wait()
end
