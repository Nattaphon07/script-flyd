local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("PSC HUB", "DarkTheme")
local Tab = Window:NewTab("Main")
local Section = Tab:NewSection("Credit : 03s. #4358")
local Tab = Window:NewTab("Auto Fram")
local Section = Tab:NewSection("Auto Frame")

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local rootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")
local userInputService = game:GetService("UserInputService")
local runService = game:GetService("RunService")

local flying = false
local invincible = false -- สถานะการเป็นอมตะ
local noAttack = false -- สถานะป้องกันการโจมตี
local speed = 50
local bodyVelocity
local bodyGyro
local direction = Vector3.new(0, 0, 0) -- ทิศทางการบิน

-- ฟังก์ชันเริ่มการบิน
local function startFlying()
    flying = true
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(1e4, 1e4, 1e4)
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.Parent = rootPart

    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(1e4, 1e4, 1e4)
    bodyGyro.CFrame = rootPart.CFrame
    bodyGyro.Parent = rootPart
end

-- ฟังก์ชันหยุดการบิน
local function stopFlying()
    flying = false
    if bodyVelocity then
        bodyVelocity:Destroy()
    end
    if bodyGyro then
        bodyGyro:Destroy()
    end
end

-- อัปเดตทิศทางการบินตามปุ่มที่ถูกกด
local function updateDirection()
    local newDirection = Vector3.new(0, 0, 0)

    if userInputService:IsKeyDown(Enum.KeyCode.W) then
        newDirection = newDirection + (rootPart.CFrame.LookVector * speed)
    end
    if userInputService:IsKeyDown(Enum.KeyCode.S) then
        newDirection = newDirection - (rootPart.CFrame.LookVector * speed)
    end
    if userInputService:IsKeyDown(Enum.KeyCode.A) then
        newDirection = newDirection - (rootPart.CFrame.RightVector * speed)
    end
    if userInputService:IsKeyDown(Enum.KeyCode.D) then
        newDirection = newDirection + (rootPart.CFrame.RightVector * speed)
    end
    if userInputService:IsKeyDown(Enum.KeyCode.E) then -- ปุ่มสำหรับบินขึ้น
        newDirection = newDirection + Vector3.new(0, speed, 0)
    end
    if userInputService:IsKeyDown(Enum.KeyCode.Q) then -- ปุ่มสำหรับบินลง
        newDirection = newDirection - Vector3.new(0, speed, 0)
    end

    direction = newDirection
end

-- ฟังก์ชันทำให้ตัวละครเลือดไม่ลด
local function preventDamage()
    if invincible then
        humanoid.Health = humanoid.MaxHealth -- ทำให้พลังชีวิตเต็มเสมอ
    end
end

-- ฟังก์ชันป้องกันการโจมตี
local function preventAttack(damage)
    if noAttack then
        humanoid.Health = humanoid.MaxHealth -- ป้องกันการลดพลังชีวิตจากการโจมตี
    end
end

-- ตรวจจับการเปลี่ยนแปลงเลือดเพื่อไม่ให้เลือดลด
humanoid.HealthChanged:Connect(function()
    preventDamage()
end)

-- ตรวจจับการเคลื่อนไหวในขณะที่บิน
runService.RenderStepped:Connect(function()
    if flying and bodyVelocity then
        updateDirection()
        bodyVelocity.Velocity = direction
        bodyGyro.CFrame = workspace.CurrentCamera.CFrame -- หมุนตัวตามกล้อง
    end
end)

-- Toggle for flying in Auto Frame section
Section:NewToggle("บินอิสระ", "ToggleInfo", function(state)
    if state then
        startFlying()
        invincible = true -- ทำให้ตัวละครเป็นอมตะ
        noAttack = true -- ป้องกันการโจมตี
    else
        stopFlying()
        invincible = false -- ปิดการเป็นอมตะ
        noAttack = false -- ปิดการป้องกันการโจมตี
    end
end)
