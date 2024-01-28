local Ball = workspace.Balls -- Assuming a single ball in workspace.Balls
local OldPosition = Ball.Position

Ball:GetPropertyChangedSignal("Position"):Connect(function()
    local ballData = CheckBall()
    if ballData[1] and ballData[3] == game.Players.LocalPlayer.Name then
        local ball = ballData[2]

        local distance = (ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
        local velocity = (OldPosition - ball.Position).Magnitude

        print(`Distance: {distance}\nVelocity: {velocity}\nTime: {distance / velocity}`)

        if (distance / velocity) <= 10 then -- Adjust threshold as needed
            game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
        end
    end

    OldPosition = Ball.Position -- Update for next calculation
end)

function CheckBall()
    for i, v in pairs(workspace.Balls:GetChildren()) do
        if v:GetAttribute("target") ~= "" then
            return {true, v, v:GetAttribute("target")}
        end
    end
    return {false}
end
