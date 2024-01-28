function CheckBall()
    for i, ball in pairs(workspace.Balls:GetChildren()) do
        local targetPlayer = ball:GetAttribute("target")
        if targetPlayer ~= "" then
            return {true, ball, targetPlayer, ball.Velocity.Magnitude}
        end
    end
    return {false}
end

while true do
    wait(0.002)
    local ballData = CheckBall()
    if ballData[1] and ballData[3] == game.Players.LocalPlayer.Name then
        local ball = ballData[2]
        local distance = (ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
        local velocity = (ball.Position - ballData[2].Position).Magnitude
        local requiredTime = distance / velocity

        print(string.format("Distance: %f\nVelocity: %f\nTime: %f", distance, velocity, requiredTime))

        if requiredTime <= 10 then
            game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
        end
    end
end
