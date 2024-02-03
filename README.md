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
    task.wait(0.002)
    local ballData = CheckBall()
    if ballData[1] and ballData[3] == game.Players.LocalPlayer.Name then
        local ball = ballData[2]
        local distance = game.Players.LocalPlayer:DistanceFromCharacter(ball.Position)
        local velocity = ball.Velocity

        -- คำนวณค่า velocityMagnitude ตาม Velocity ของลูก
        local velocityMagnitude = math.sqrt(velocity.X^2 + velocity.Y^2 + velocity.Z^2)

        if distance <= (velocityMagnitude / 1.5) then
            game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
            print("HavokAI: ParryButtonPress event fired. Prepare for chaos!")
        end
    end
end
