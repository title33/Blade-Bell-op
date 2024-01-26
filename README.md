getgenv().god = true

while getgenv().god and task.wait() do
    local localPlayer = game.Players.LocalPlayer
    local character = localPlayer.Character

    if character and character:FindFirstChild("HumanoidRootPart") then
        local humanoidRootPart = character.HumanoidRootPart

        for _, ball in ipairs(workspace.Balls:GetChildren()) do
            if ball then
               
                local direction = (ball.Position - humanoidRootPart.Position).unit

                
                humanoidRootPart.CFrame = CFrame.lookAt(humanoidRootPart.Position, ball.Position)

                if character:FindFirstChild("Highlight") then
                    
                    humanoidRootPart.CFrame = CFrame.new(humanoidRootPart.Position, ball.Position) * CFrame.new(0, 0, (ball.Velocity).Magnitude * -0.5)
                    game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
                end
            end
        end
    end
end
