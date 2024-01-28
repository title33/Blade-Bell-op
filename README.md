function PredictPosition(ball, time)
    -- ทำนายตำแหน่งโดยใช้ Velocity และเวลาที่กำหนด
    return ball.Position + ball.Velocity * time
end

function CheckBall()
    for _, ball in pairs(workspace.Balls:GetChildren()) do
        local targetPlayer = ball:GetAttribute("target")
        if targetPlayer ~= "" then
            local requiredDistance = ball.Velocity.Magnitude / 1.5
            return ball, targetPlayer, requiredDistance
        end
    end
    return nil
end

while true do
    wait(0.001)  -- ลดระยะเวลา wait เล็กน้อย
    local ball = CheckBall()
    if ball and ball[2] == game.Players.LocalPlayer.Name then
        local distance = game.Players.LocalPlayer:DistanceFromCharacter(ball[1].Position)
        local requiredTime = distance / ball[1].Velocity.Magnitude
        local predictedPosition = PredictPosition(ball[1], requiredTime)

        if (game.Players.LocalPlayer:DistanceFromCharacter(predictedPosition) <= ball[3]) then
            game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
        end
    end
end
