local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ProfessionalMenu"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

-- Painel principal
local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 420, 0, 260)
Main.Position = UDim2.new(0.5, -210, 0.5, -130)
Main.BackgroundColor3 = Color3.fromRGB(25,25,25)
Main.BorderSizePixel = 0
Main.Parent = ScreenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0,12)
corner.Parent = Main

local shadow = Instance.new("UIStroke")
shadow.Thickness = 1
shadow.Color = Color3.fromRGB(50,50,50)
shadow.Parent = Main

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,40)
Title.BackgroundTransparency = 1
Title.Text = "Painel de Ferramenta"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.TextColor3 = Color3.fromRGB(240,240,240)
Title.Parent = Main

-- TextBox
local TextBox = Instance.new("TextBox")
TextBox.Size = UDim2.new(0.8,0,0,40)
TextBox.Position = UDim2.new(0.1,0,0.35,0)
TextBox.BackgroundColor3 = Color3.fromRGB(35,35,35)
TextBox.TextColor3 = Color3.fromRGB(220,220,220)
TextBox.PlaceholderText = "Digite um comando..."
TextBox.Font = Enum.Font.Gotham
TextBox.TextSize = 16
TextBox.BorderSizePixel = 0
TextBox.Parent = Main

local corner2 = Instance.new("UICorner")
corner2.CornerRadius = UDim.new(0,8)
corner2.Parent = TextBox

-- Botão principal
local Button = Instance.new("TextButton")
Button.Size = UDim2.new(0.5,0,0,45)
Button.Position = UDim2.new(0.25,0,0.65,0)
Button.BackgroundColor3 = Color3.fromRGB(0,120,255)
Button.Text = "Executar"
Button.TextColor3 = Color3.new(1,1,1)
Button.Font = Enum.Font.GothamBold
Button.TextSize = 16
Button.BorderSizePixel = 0
Button.Parent = Main

local corner3 = Instance.new("UICorner")
corner3.CornerRadius = UDim.new(0,8)
corner3.Parent = Button

-- Hover Effect
Button.MouseEnter:Connect(function()
	TweenService:Create(
		Button,
		TweenInfo.new(0.2),
		{BackgroundColor3 = Color3.fromRGB(0,150,255)}
	):Play()
end)

Button.MouseLeave:Connect(function()
	TweenService:Create(
		Button,
		TweenInfo.new(0.2),
		{BackgroundColor3 = Color3.fromRGB(0,120,255)}
	):Play()
end)

-- Função botão
Button.MouseButton1Click:Connect(function()
	print("Comando:", TextBox.Text)
end)

-- Animação de abertura
Main.Size = UDim2.new(0,0,0,0)

TweenService:Create(
	Main,
	TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
	{Size = UDim2.new(0,420,0,260)}
):Play()

-- Toggle com tecla RightShift
local UIS = game:GetService("UserInputService")

UIS.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	
	if input.KeyCode == Enum.KeyCode.RightShift then
		
		if Main.Visible then
			
			local closeTween = TweenService:Create(
				Main,
				TweenInfo.new(0.25),
				{Size = UDim2.new(0,0,0,0)}
			)
			
			closeTween:Play()
			
			closeTween.Completed:Connect(function()
				Main.Visible = false
			end)
			
		else
			
			Main.Visible = true
			
			TweenService:Create(
				Main,
				TweenInfo.new(0.25),
				{Size = UDim2.new(0,420,0,260)}
			):Play()
			
		end
		
	end
end)
