
-- coder: olz
local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")

local flying = false
local speed = 50
local bv, bg
local minimized = false -- Controle do estado do menu

local sg = Instance.new("ScreenGui", player.PlayerGui)
sg.Name = "olz_fly_v5"
sg.ResetOnSpawn = false

local frame = Instance.new("Frame", sg)
frame.Size = UDim2.new(0, 160, 0, 180)
frame.Position = UDim2.new(0.1, 0, 0.4, 0)
frame.BackgroundColor3 = Color3.new(0, 0, 0)
frame.BorderSizePixel = 2
frame.BorderColor3 = Color3.fromHex("770000")
frame.Active = true
frame.Draggable = true 

local closeBtn = Instance.new("TextButton", frame)
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -30, 0, 0)
closeBtn.BackgroundColor3 = Color3.fromHex("770000")
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.MouseButton1Click:Connect(function() sg:Destroy() end)

-- Botão Minimizar
local minBtn = Instance.new("TextButton", frame)
minBtn.Size = UDim2.new(0, 30, 0, 30)
minBtn.Position = UDim2.new(1, -62, 0, 0) minBtn.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
minBtn.Text = "-"
minBtn.TextColor3 = Color3.new(1, 1, 1)
minBtn.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(0.6, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "coder: olz"
title.Font = Enum.Font.Code
title.TextSize = 16

local controlsFrame = Instance.new("Frame", frame)
controlsFrame.Size = UDim2.new(1, 0, 1, -30)
controlsFrame.Position = UDim2.new(0, 0, 0, 30)
controlsFrame.BackgroundTransparency = 1

local flyBtn = Instance.new("TextButton", controlsFrame)
flyBtn.Size = UDim2.new(0.9, 0, 0, 35)
flyBtn.Position = UDim2.new(0.05, 0, 0.1, 0)
flyBtn.BackgroundColor3 = Color3.fromHex("770000")
flyBtn.TextColor3 = Color3.new(1, 1, 1)
flyBtn.Text = "fly: off"

local speedLabel = Instance.new("TextLabel", controlsFrame)
speedLabel.Size = UDim2.new(1, 0, 0, 20)
speedLabel.Position = UDim2.new(0, 0, 0.4, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.Text = "velocidade: " .. speed

local addSpeed = Instance.new("TextButton", controlsFrame)
addSpeed.Size = UDim2.new(0.4, 0, 0, 35)
addSpeed.Position = UDim2.new(0.05, 0, 0.6, 0)
addSpeed.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
addSpeed.TextColor3 = Color3.new(1, 1, 1)
addSpeed.Text = "+"

local subSpeed = Instance.new("TextButton", controlsFrame)
subSpeed.Size = UDim2.new(0.4, 0, 0, 35)
subSpeed.Position = UDim2.new(0.55, 0, 0.6, 0)
subSpeed.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
subSpeed.TextColor3 = Color3.new(1, 1, 1)
subSpeed.Text = "-"

minBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        frame:TweenSize(UDim2.new(0, 160, 0, 30), "Out", "Quad", 0.3, true)
        controlsFrame.Visible = false
        minBtn.Text = "+"
    else
        frame:TweenSize(UDim2.new(0, 160, 0, 180), "Out", "Quad", 0.3, true)
        controlsFrame.Visible = true
        minBtn.Text = "-"
    end
end)

runService.RenderStepped:Connect(function()
    title.TextColor3 = Color3.fromHSV(tick() % 3 / 3, 1, 1)
end)

addSpeed.MouseButton1Click:Connect(function()
    speed = speed + 1
    speedLabel.Text = "velocidade: " .. speed
end)

subSpeed.MouseButton1Click:Connect(function()
    speed = math.max(0, speed - 1)
    speedLabel.Text = "velocidade: " .. speed
end)

flyBtn.MouseButton1Click:Connect(function()
    flying = not flying
    local char = player.Character
    if flying and char then
        flyBtn.Text = "fly: on"
        local hrp = char:WaitForChild("HumanoidRootPart")
        bv = Instance.new("BodyVelocity", hrp)
        bg = Instance.new("BodyGyro", hrp)
        bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        char.Humanoid.PlatformStand = true
    else
        flyBtn.Text = "fly: off"
        if bv then bv:Destroy() end
        if bg then bg:Destroy() end
        if char then char.Humanoid.PlatformStand = false end
    end
end)

runService.RenderStepped:Connect(function()
    if flying and bv and bg and player.Character then
        local cam = workspace.CurrentCamera
        local hum = player.Character:FindFirstChildOfClass("Humanoid")
        local moveDir = hum.MoveDirection
        
        if moveDir.Magnitude > 0 then
            local directionFactor = moveDir:Dot(cam.CFrame.LookVector)
            if directionFactor > 0 then
                bv.Velocity = cam.CFrame.LookVector * speed
            else
                bv.Velocity = cam.CFrame.LookVector * -speed
            end
        else
            bv.Velocity = Vector3.new(0, 0.1, 0)
        end
        bg.CFrame = cam.CFrame
    end
end)
