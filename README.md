function CheckBall()
    for i, ball in pairs(workspace.Balls:GetChildren()) do
        local targetPlayer = ball:GetAttribute("target")
        if targetPlayer ~= "" then
            return {true, ball, targetPlayer, ball.Velocity}
        end
    end
    return {false}
end

while true do
    task.wait(0.001)  -- ลดระยะเวลารอลง
    local ballData = CheckBall()

    if ballData[1] and ballData[3] == game.Players.LocalPlayer.Name then
        local ball = ballData[2]
        local distance = game.Players.LocalPlayer:DistanceFromCharacter(ball.Position)
        local initialVelocity = ball.Velocity

        -- รอเพียง 0.01 วินาที
        task.wait(0.01)

        -- คำนวณความเร็วใหม่ที่ได้จากการเปลี่ยนแปลง
        local updatedVelocity = ball.Velocity
        local velocityChange = updatedVelocity - initialVelocity
        local velocityMagnitudeChange = velocityChange.Magnitude

        -- คำนวณความเร็วของลูกบอล
        local velocityMagnitude = updatedVelocity.Magnitude

        -- ตรวจสอบการ Parry โดยใช้ความเร็วของลูกบอลที่ถูกคำนวณใหม่
        if distance <= (velocityMagnitude / 1.5) and velocityMagnitudeChange >= 50 then
            game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
            print("Xylo: ParryButtonPress event fired. Prepare for chaos!")
        end
    end
end
