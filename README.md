while true do
    wait(0.002)
    local ballData = CheckBall()

    if ballData[1] and ballData[3] == game.Players.LocalPlayer.Name then
        local ball = ballData[2]
        local distance = game.Players.LocalPlayer:DistanceFromCharacter(ball.Position)
        local velocity = ball.Velocity.Magnitude

        -- Calculate the threshold dynamically based on velocity
        local parryThreshold = velocity * 0.1 -- Adjust the multiplier as needed

        -- Calculate the predicted position of the ball after a short duration (0.1 seconds in this case)
        local predictedPosition = ball.Position + ball.Velocity * 0.1
        local predictedDistance = (predictedPosition - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude

        if predictedDistance <= parryThreshold then
            -- Ensure that the player is facing the predicted position of the ball before triggering Parry
            local playerDirection = (predictedPosition - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Unit
            local playerForward = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame.LookVector

            local dotProduct = playerDirection:Dot(playerForward)
            local angle = math.acos(dotProduct)

            -- Set the angle threshold based on your preference (in radians)
            local angleThreshold = math.rad(30) -- Adjust the angle threshold as needed

            if angle <= angleThreshold then
                -- Player is facing the predicted position of the ball, trigger Parry
                game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
                print("Xylo hub")
            end
        end
    end
end
