local Ball = Instance.new("Part")
Ball.Size = Vector3.new(5, 5, 5)
Ball.Shape = Enum.PartType.Ball
Ball.Material = Enum.Material.ForceField
Ball.CanQuery = false
Ball.CanTouch = false
Ball.CanCollide = false
Ball.CastShadow = false
Ball.Color = Color3.fromRGB(255, 255, 255)
Ball.Parent = workspace

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

while true do
    wait()
    if character and character.PrimaryPart then  -- ตรวจสอบว่าตัวละครพร้อมใช้งาน
        Ball.CFrame = character.PrimaryPart.CFrame  -- ตั้งค่า CFrame ของลูกบอลให้เท่ากับ CFrame ของตัวละคร
    end
end
