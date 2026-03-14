--[[

	Aimbot Module [AirHub] by Exunys © CC0 1.0 Universal (2023)

	https://github.com/Exunys

]]

--// Cache

local pcall, getgenv, next, setmetatable, Vector2new, CFramenew, Color3fromRGB, Drawingnew, TweenInfonew, stringupper, mousemoverel = pcall, getgenv, next, setmetatable, Vector2.new, CFrame.new, Color3.fromRGB, Drawing.new, TweenInfo.new, string.upper, mousemoverel or (Input and Input.MouseMove)

--// Launching checks

if not getgenv().AirHub or getgenv().AirHub.Aimbot then return end

--// Services

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// Variables

local RequiredDistance, Typing, Running, ServiceConnections, Animation, OriginalSensitivity = 2000, false, false, {}

--// Environment

getgenv().AirHub.Aimbot = {
	Settings = {
		Enabled = false,
		TeamCheck = false,
		AliveCheck = true,
		WallCheck = false,
		Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
		ThirdPerson = false, -- Uses mousemoverel instead of CFrame to support locking in third person (could be choppy)
		ThirdPersonSensitivity = 3,
		TriggerKey = "MouseButton2",
		Toggle = false,
		LockPart = "Head" -- Body part to lock on
	},

	FOVSettings = {
		Enabled = true,
		Visible = true,
		Amount = 90,
		Color = Color3fromRGB(255, 255, 255),
		LockedColor = Color3fromRGB(255, 70, 70),
		Transparency = 0.5,
		Sides = 60,
		Thickness = 1,
		Filled = false
	},

	FOVCircle = Drawingnew("Circle")
}

local Environment = getgenv().AirHub.Aimbot

--// Core Functions

local function ConvertVector(Vector)
	return Vector2new(Vector.X, Vector.Y)
end

local function CancelLock()
	Environment.Locked = nil
	Environment.FOVCircle.Color = Environment.FOVSettings.Color
	UserInputService.MouseDeltaSensitivity = OriginalSensitivity

	if Animation then
		Animation:Cancel()
	end
end

local function GetClosestPlayer()
	if not Environment.Locked then
		RequiredDistance = (Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000)

		for _, v in next, Players:GetPlayers() do
			if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(Environment.Settings.LockPart) and v.Character:FindFirstChildOfClass("Humanoid") then
				if Environment.Settings.TeamCheck and v.TeamColor == LocalPlayer.TeamColor then continue end
				if Environment.Settings.AliveCheck and v.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then continue end
				if Environment.Settings.WallCheck and #(Camera:GetPartsObscuringTarget({v.Character[Environment.Settings.LockPart].Position}, v.Character:GetDescendants())) > 0 then continue end

				local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position); Vector = ConvertVector(Vector)
				local Distance = (UserInputService:GetMouseLocation() - Vector).Magnitude

				if Distance < RequiredDistance and OnScreen then
					RequiredDistance = Distance
					Environment.Locked = v
				end
			end
		end
	elseif (UserInputService:GetMouseLocation() - ConvertVector(Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position))).Magnitude > RequiredDistance then
		CancelLock()
	end
end

local function Load()
	OriginalSensitivity = UserInputService.MouseDeltaSensitivity

	ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
		if Environment.FOVSettings.Enabled and Environment.Settings.Enabled then
			Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
			Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
			Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
			Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
			Environment.FOVCircle.Color = Environment.FOVSettings.Color
			Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
			Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
			Environment.FOVCircle.Position = Vector2new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
		else
			Environment.FOVCircle.Visible = false
		end

		if Running and Environment.Settings.Enabled then
			GetClosestPlayer()

			if Environment.Locked then
				if Environment.Settings.ThirdPerson then
					local Vector = Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position)

					mousemoverel((Vector.X - UserInputService:GetMouseLocation().X) * Environment.Settings.ThirdPersonSensitivity, (Vector.Y - UserInputService:GetMouseLocation().Y) * Environment.Settings.ThirdPersonSensitivity)
				else
					if Environment.Settings.Sensitivity > 0 then
						Animation = TweenService:Create(Camera, TweenInfonew(Environment.Settings.Sensitivity, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFramenew(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)})
						Animation:Play()
					else
						Camera.CFrame = CFramenew(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)
					end

					UserInputService.MouseDeltaSensitivity = 0
				end

				Environment.FOVCircle.Color = Environment.FOVSettings.LockedColor
			end
		end
	end)

	ServiceConnections.InputBeganConnection = UserInputService.InputBegan:Connect(function(Input)
		if not Typing then
			pcall(function()
				if Input.UserInputType == Enum.UserInputType.Keyboard and Input.KeyCode == Enum.KeyCode[#Environment.Settings.TriggerKey == 1 and stringupper(Environment.Settings.TriggerKey) or Environment.Settings.TriggerKey] or Input.UserInputType == Enum.UserInputType[Environment.Settings.TriggerKey] then
					if Environment.Settings.Toggle then
						Running = not Running

						if not Running then
							CancelLock()
						end
					else
						Running = true
					end
				end
			end)
		end
	end)

	ServiceConnections.InputEndedConnection = UserInputService.InputEnded:Connect(function(Input)
		if not Typing then
			if not Environment.Settings.Toggle then
				pcall(function()
					if Input.UserInputType == Enum.UserInputType.Keyboard and Input.KeyCode == Enum.KeyCode[#Environment.Settings.TriggerKey == 1 and stringupper(Environment.Settings.TriggerKey) or Environment.Settings.TriggerKey] or Input.UserInputType == Enum.UserInputType[Environment.Settings.TriggerKey] then
						Running = false; CancelLock()
					end
				end)
			end
		end
	end)
end

--// Typing Check

ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
	Typing = true
end)

ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
	Typing = false
end)

--// Functions

Environment.Functions = {}

function Environment.Functions:Exit()
	for _, v in next, ServiceConnections do
		v:Disconnect()
	end

	Environment.FOVCircle:Remove()

	getgenv().AirHub.Aimbot.Functions = nil
	getgenv().AirHub.Aimbot = nil

	Load = nil; ConvertVector = nil; CancelLock = nil; GetClosestPlayer = nil;
end

function Environment.Functions:Restart()
	for _, v in next, ServiceConnections do
		v:Disconnect()
	end

	Load()
end

function Environment.Functions:ResetSettings()
	Environment.Settings = {
		Enabled = false,
		TeamCheck = false,
		AliveCheck = true,
		WallCheck = false,
		Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
		ThirdPerson = false, -- Uses mousemoverel instead of CFrame to support locking in third person (could be choppy)
		ThirdPersonSensitivity = 3,
		TriggerKey = "MouseButton2",
		Toggle = false,
		LockPart = "Head" -- Body part to lock on
	}

	Environment.FOVSettings = {
		Enabled = true,
		Visible = true,
		Amount = 90,
		Color = Color3fromRGB(255, 255, 255),
		LockedColor = Color3fromRGB(255, 70, 70),
		Transparency = 0.5,
		Sides = 60,
		Thickness = 1,
		Filled = false
	}
end

setmetatable(Environment.Functions, {
	__newindex = warn
})

--// Load

Load()


--[[

	Wall Hack Module [AirHub] by Exunys © CC0 1.0 Universal (2023)

	https://github.com/Exunys

]]

--// Cache

local select, next, tostring, pcall, getgenv, setmetatable, mathfloor, mathabs, mathcos, mathsin, mathrad, wait = select, next, tostring, pcall, getgenv, setmetatable, math.floor, math.abs, math.cos, math.sin, math.rad, task.wait
local WorldToViewportPoint, Vector2new, Vector3new, Vector3zero, CFramenew, Drawingnew, Color3fromRGB = nil, Vector2.new, Vector3.new, Vector3.zero, CFrame.new, Drawing.new, Color3.fromRGB

--// Launching checks

if not getgenv().AirHub or getgenv().AirHub.WallHack then return end

--// Services

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// Variables

local ServiceConnections = {}

--// Environment

getgenv().AirHub.WallHack = {
	Settings = {
		Enabled = false,
		TeamCheck = false,
		AliveCheck = true
	},

	Visuals = {
		ChamsSettings = {
			Enabled = false,
			Color = Color3fromRGB(255, 255, 255),
			Transparency = 0.2,
			Thickness = 0,
			Filled = true,
			EntireBody = false -- For R15, keep false to prevent lag
		},

		ESPSettings = {
			Enabled = true,
			TextColor = Color3fromRGB(255, 255, 255),
			TextSize = 14,
			Outline = true,
			OutlineColor = Color3fromRGB(0, 0, 0),
			TextTransparency = 0.7,
			TextFont = Drawing.Fonts.UI, -- UI, System, Plex, Monospace
			Offset = 20,
			DisplayDistance = true,
			DisplayHealth = true,
			DisplayName = true
		},

		TracersSettings = {
			Enabled = true,
			Type = 1, -- 1 - Bottom; 2 - Center; 3 - Mouse
			Transparency = 0.7,
			Thickness = 1,
			Color = Color3fromRGB(255, 255, 255)
		},

		BoxSettings = {
			Enabled = true,
			Type = 1; -- 1 - 3D; 2 - 2D
			Color = Color3fromRGB(255, 255, 255),
			Transparency = 0.7,
			Thickness = 1,
			Filled = false, -- For 2D
			Increase = 1 -- For 3D
		},

		HeadDotSettings = {
			Enabled = true,
			Color = Color3fromRGB(255, 255, 255),
			Transparency = 0.5,
			Thickness = 1,
			Filled = false,
			Sides = 30
		},

		HealthBarSettings = {
			Enabled = false,
			Transparency = 0.8,
			Size = 2,
			Offset = 10,
			OutlineColor = Color3fromRGB(0, 0, 0),
			Blue = 50,
			Type = 3 -- 1 - Top; 2 - Bottom; 3 - Left; 4 - Right
		}
	},

	Crosshair = {
		Settings = {
			Enabled = false,
			Type = 1, -- 1 - Mouse; 2 - Center
			Size = 12,
			Thickness = 1,
			Color = Color3fromRGB(0, 255, 0),
			Transparency = 1,
			GapSize = 5,
			Rotation = 0,
			CenterDot = false,
			CenterDotColor = Color3fromRGB(0, 255, 0),
			CenterDotSize = 1,
			CenterDotTransparency = 1,
			CenterDotFilled = true,
			CenterDotThickness = 1
		},

		Parts = {
			LeftLine = Drawingnew("Line"),
			RightLine = Drawingnew("Line"),
			TopLine = Drawingnew("Line"),
			BottomLine = Drawingnew("Line"),
			CenterDot = Drawingnew("Circle")
		}
	},

	WrappedPlayers = {}
}

local Environment = getgenv().AirHub.WallHack

--// Core Functions

WorldToViewportPoint = function(...)
	return Camera.WorldToViewportPoint(Camera, ...)
end

local function GetPlayerTable(Player)
	for _, v in next, Environment.WrappedPlayers do
		if v.Name == Player.Name then
			return v
		end
	end
end

local function AssignRigType(Player)
	local PlayerTable = GetPlayerTable(Player)

	repeat wait(0) until Player.Character

	if Player.Character:FindFirstChild("Torso") and not Player.Character:FindFirstChild("LowerTorso") then
		PlayerTable.RigType = "R6"
	elseif Player.Character:FindFirstChild("LowerTorso") and not Player.Character:FindFirstChild("Torso") then
		PlayerTable.RigType = "R15"
	else
		repeat AssignRigType(Player) until PlayerTable.RigType
	end
end

local function InitChecks(Player)
	local PlayerTable = GetPlayerTable(Player)

	PlayerTable.Connections.UpdateChecks = RunService.RenderStepped:Connect(function()
		if Player.Character and Player.Character:FindFirstChildOfClass("Humanoid") then
			if Environment.Settings.AliveCheck then
				PlayerTable.Checks.Alive = Player.Character:FindFirstChildOfClass("Humanoid").Health > 0
			else
				PlayerTable.Checks.Alive = true
			end

			if Environment.Settings.TeamCheck then
				PlayerTable.Checks.Team = Player.TeamColor ~= LocalPlayer.TeamColor
			else
				PlayerTable.Checks.Team = true
			end
		else
			PlayerTable.Checks.Alive = false
			PlayerTable.Checks.Team = false
		end
	end)
end

local function UpdateCham(Part, Cham)
	local CorFrame, PartSize = Part.CFrame, Part.Size / 2

	if select(2, WorldToViewportPoint(CorFrame * CFramenew(PartSize.X / 2,  PartSize.Y / 2, PartSize.Z / 2).Position)) and Environment.Visuals.ChamsSettings.Enabled then

		--// Quad 1 - Front

		Cham.Quad1.Transparency = Environment.Visuals.ChamsSettings.Transparency
		Cham.Quad1.Color = Environment.Visuals.ChamsSettings.Color
		Cham.Quad1.Thickness = Environment.Visuals.ChamsSettings.Thickness
		Cham.Quad1.Filled = Environment.Visuals.ChamsSettings.Filled
		Cham.Quad1.Visible = Environment.Visuals.ChamsSettings.Enabled

		local PosTopLeft = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X,  PartSize.Y, PartSize.Z).Position)
		local PosTopRight = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X,  PartSize.Y, PartSize.Z).Position)
		local PosBottomLeft = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X, -PartSize.Y, PartSize.Z).Position)
		local PosBottomRight = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, -PartSize.Y, PartSize.Z).Position)

		Cham.Quad1.PointA = Vector2new(PosTopLeft.X, PosTopLeft.Y)
		Cham.Quad1.PointB = Vector2new(PosBottomLeft.X, PosBottomLeft.Y)
		Cham.Quad1.PointC = Vector2new(PosBottomRight.X, PosBottomRight.Y)
		Cham.Quad1.PointD = Vector2new(PosTopRight.X, PosTopRight.Y)

		--// Quad 2 - Back

		Cham.Quad2.Transparency = Environment.Visuals.ChamsSettings.Transparency
		Cham.Quad2.Color = Environment.Visuals.ChamsSettings.Color
		Cham.Quad2.Thickness = Environment.Visuals.ChamsSettings.Thickness
		Cham.Quad2.Filled = Environment.Visuals.ChamsSettings.Filled
		Cham.Quad2.Visible = Environment.Visuals.ChamsSettings.Enabled

		local PosTopLeft2 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X,  PartSize.Y, -PartSize.Z).Position)
		local PosTopRight2 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X,  PartSize.Y, -PartSize.Z).Position)
		local PosBottomLeft2 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X, -PartSize.Y, -PartSize.Z).Position)
		local PosBottomRight2 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, -PartSize.Y, -PartSize.Z).Position)

		Cham.Quad2.PointA = Vector2new(PosTopLeft2.X, PosTopLeft2.Y)
		Cham.Quad2.PointB = Vector2new(PosBottomLeft2.X, PosBottomLeft2.Y)
		Cham.Quad2.PointC = Vector2new(PosBottomRight2.X, PosBottomRight2.Y)
		Cham.Quad2.PointD = Vector2new(PosTopRight2.X, PosTopRight2.Y)

		--// Quad 3 - Top

		Cham.Quad3.Transparency = Environment.Visuals.ChamsSettings.Transparency
		Cham.Quad3.Color = Environment.Visuals.ChamsSettings.Color
		Cham.Quad3.Thickness = Environment.Visuals.ChamsSettings.Thickness
		Cham.Quad3.Filled = Environment.Visuals.ChamsSettings.Filled
		Cham.Quad3.Visible = Environment.Visuals.ChamsSettings.Enabled

		local PosTopLeft3 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X,  PartSize.Y, PartSize.Z).Position)
		local PosTopRight3 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, PartSize.Y, PartSize.Z).Position)
		local PosBottomLeft3 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X, PartSize.Y, -PartSize.Z).Position)
		local PosBottomRight3 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, PartSize.Y, -PartSize.Z).Position)

		Cham.Quad3.PointA = Vector2new(PosTopLeft3.X, PosTopLeft3.Y)
		Cham.Quad3.PointB = Vector2new(PosBottomLeft3.X, PosBottomLeft3.Y)
		Cham.Quad3.PointC = Vector2new(PosBottomRight3.X, PosBottomRight3.Y)
		Cham.Quad3.PointD = Vector2new(PosTopRight3.X, PosTopRight3.Y)

		--// Quad 4 - Bottom

		Cham.Quad4.Transparency = Environment.Visuals.ChamsSettings.Transparency
		Cham.Quad4.Color = Environment.Visuals.ChamsSettings.Color
		Cham.Quad4.Thickness = Environment.Visuals.ChamsSettings.Thickness
		Cham.Quad4.Filled = Environment.Visuals.ChamsSettings.Filled
		Cham.Quad4.Visible = Environment.Visuals.ChamsSettings.Enabled

		local PosTopLeft4 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X,  -PartSize.Y, PartSize.Z).Position)
		local PosTopRight4 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, -PartSize.Y, PartSize.Z).Position)
		local PosBottomLeft4 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X, -PartSize.Y, -PartSize.Z).Position)
		local PosBottomRight4 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, -PartSize.Y, -PartSize.Z).Position)

		Cham.Quad4.PointA = Vector2new(PosTopLeft4.X, PosTopLeft4.Y)
		Cham.Quad4.PointB = Vector2new(PosBottomLeft4.X, PosBottomLeft4.Y)
		Cham.Quad4.PointC = Vector2new(PosBottomRight4.X, PosBottomRight4.Y)
		Cham.Quad4.PointD = Vector2new(PosTopRight4.X, PosTopRight4.Y)

		--// Quad 5 - Right

		Cham.Quad5.Transparency = Environment.Visuals.ChamsSettings.Transparency
		Cham.Quad5.Color = Environment.Visuals.ChamsSettings.Color
		Cham.Quad5.Thickness = Environment.Visuals.ChamsSettings.Thickness
		Cham.Quad5.Filled = Environment.Visuals.ChamsSettings.Filled
		Cham.Quad5.Visible = Environment.Visuals.ChamsSettings.Enabled

		local PosTopLeft5 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X,  PartSize.Y, PartSize.Z).Position)
		local PosTopRight5 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X, PartSize.Y, -PartSize.Z).Position)
		local PosBottomLeft5 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X, -PartSize.Y, PartSize.Z).Position)
		local PosBottomRight5 = WorldToViewportPoint(CorFrame * CFramenew(PartSize.X, -PartSize.Y, -PartSize.Z).Position)

		Cham.Quad5.PointA = Vector2new(PosTopLeft5.X, PosTopLeft5.Y)
		Cham.Quad5.PointB = Vector2new(PosBottomLeft5.X, PosBottomLeft5.Y)
		Cham.Quad5.PointC = Vector2new(PosBottomRight5.X, PosBottomRight5.Y)
		Cham.Quad5.PointD = Vector2new(PosTopRight5.X, PosTopRight5.Y)

		--// Quad 6 - Left

		Cham.Quad6.Transparency = Environment.Visuals.ChamsSettings.Transparency
		Cham.Quad6.Color = Environment.Visuals.ChamsSettings.Color
		Cham.Quad6.Thickness = Environment.Visuals.ChamsSettings.Thickness
		Cham.Quad6.Filled = Environment.Visuals.ChamsSettings.Filled
		Cham.Quad6.Visible = Environment.Visuals.ChamsSettings.Enabled

		local PosTopLeft6 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X,  PartSize.Y, PartSize.Z).Position)
		local PosTopRight6 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, PartSize.Y, -PartSize.Z).Position)
		local PosBottomLeft6 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, -PartSize.Y, PartSize.Z).Position)
		local PosBottomRight6 = WorldToViewportPoint(CorFrame * CFramenew(-PartSize.X, -PartSize.Y, -PartSize.Z).Position)

		Cham.Quad6.PointA = Vector2new(PosTopLeft6.X, PosTopLeft6.Y)
		Cham.Quad6.PointB = Vector2new(PosBottomLeft6.X, PosBottomLeft6.Y)
		Cham.Quad6.PointC = Vector2new(PosBottomRight6.X, PosBottomRight6.Y)
		Cham.Quad6.PointD = Vector2new(PosTopRight6.X, PosTopRight6.Y)
	else
		for i = 1, 6 do
			Cham["Quad"..tostring(i)].Visible = false
		end
	end
end

--// Visuals

local Visuals = {
	AddChams = function(Player)
		local PlayerTable = GetPlayerTable(Player)

		local function UpdateRig()
			for _, v in next, PlayerTable.Chams do
				for i = 1, 6 do
					local Quad = v["Quad"..tostring(i)]

					if Quad and Quad.Remove then
						Quad:Remove()
					end
				end
			end

			if PlayerTable.RigType == "R15" then
				if not Environment.Visuals.ChamsSettings.EntireBody then
					PlayerTable.Chams = {
						Head = {},
						UpperTorso = {},
						LeftLowerArm = {}, LeftUpperArm = {},
						RightLowerArm = {}, RightUpperArm = {},
						LeftLowerLeg = {}, LeftUpperLeg = {},
						RightLowerLeg = {}, RightUpperLeg = {}
					}
				else
					PlayerTable.Chams = {
						Head = {},
						UpperTorso = {}, LowerTorso = {},
						LeftLowerArm = {}, LeftUpperArm = {}, LeftHand = {},
						RightLowerArm = {}, RightUpperArm = {}, RightHand = {},
						LeftLowerLeg = {}, LeftUpperLeg = {}, LeftFoot = {},
						RightLowerLeg = {}, RightUpperLeg = {}, RightFoot = {}
					}
				end
			elseif PlayerTable.RigType == "R6" then
				PlayerTable.Chams = {
					Head = {},
					Torso = {},
					["Left Arm"] = {},
					["Right Arm"] = {},
					["Left Leg"] = {},
					["Right Leg"] = {}
				}
			end

			for _, v in next, PlayerTable.Chams do
				for i = 1, 6 do
					v["Quad"..tostring(i)] = Drawingnew("Quad")
				end
			end
		end

		local OldEntireBody = Environment.Visuals.ChamsSettings.EntireBody

		UpdateRig(); PlayerTable.Connections.Chams = RunService.RenderStepped:Connect(function()
			for i, v in next, PlayerTable.Chams do
				UpdateCham(Player.Character:WaitForChild(i, 1 / 0), v)
			end

			if Environment.Visuals.ChamsSettings.Enabled then
				if Environment.Visuals.ChamsSettings.EntireBody ~= OldEntireBody then
					UpdateRig(); OldEntireBody = Environment.Visuals.ChamsSettings.EntireBody
				end
			end
		end)
	end,

	AddESP = function(Player)
		local PlayerTable = GetPlayerTable(Player)

		PlayerTable.ESP = Drawingnew("Text")

		PlayerTable.Connections.ESP = RunService.RenderStepped:Connect(function()
			if Player.Character and Player.Character:FindFirstChildOfClass("Humanoid") and Player.Character:FindFirstChild("HumanoidRootPart") and Player.Character:FindFirstChild("Head") and Environment.Settings.Enabled then
				local Vector, OnScreen = WorldToViewportPoint(Player.Character.Head.Position)

				PlayerTable.ESP.Visible = Environment.Visuals.ESPSettings.Enabled

				if OnScreen and Environment.Visuals.ESPSettings.Enabled then
					PlayerTable.ESP.Visible = PlayerTable.Checks.Alive and PlayerTable.Checks.Team and true or false

					if PlayerTable.ESP.Visible then
						PlayerTable.ESP.Center = true
						PlayerTable.ESP.Size = Environment.Visuals.ESPSettings.TextSize
						PlayerTable.ESP.Outline = Environment.Visuals.ESPSettings.Outline
						PlayerTable.ESP.OutlineColor = Environment.Visuals.ESPSettings.OutlineColor
						PlayerTable.ESP.Color = Environment.Visuals.ESPSettings.TextColor
						PlayerTable.ESP.Transparency = Environment.Visuals.ESPSettings.TextTransparency
						PlayerTable.ESP.Font = Environment.Visuals.ESPSettings.TextFont

						local Parts, Content, Tool = {
							Health = "("..tostring(mathfloor(Player.Character.Humanoid.Health))..")",
							Distance = "["..tostring(mathfloor((Player.Character.HumanoidRootPart.Position or Vector3zero - (LocalPlayer.Character.HumanoidRootPart.Position or Vector3zero)).Magnitude)).."]",
							Name = Player.DisplayName == Player.Name and Player.Name or Player.DisplayName.." {"..Player.Name.."}"
						}, "", Player.Character:FindFirstChildOfClass("Tool")

						if Environment.Visuals.ESPSettings.DisplayName then
							Content = Parts.Name..Content
						end

						if Environment.Visuals.ESPSettings.DisplayHealth then
							Content = Parts.Health..(Environment.Visuals.ESPSettings.DisplayName and " " or "")..Content
						end

						if Environment.Visuals.ESPSettings.DisplayDistance then
							Content = Content.." "..Parts.Distance
						end

						PlayerTable.ESP.Text = (Tool and "["..Tool.Name.."]\n" or "")..Content
						PlayerTable.ESP.Position = Vector2new(Vector.X, Vector.Y - Environment.Visuals.ESPSettings.Offset - (Tool and 10 or 0))
					end
				else
					PlayerTable.ESP.Visible = false
				end
			else
				PlayerTable.ESP.Visible = false
			end
		end)
	end,

	AddTracer = function(Player)
		local PlayerTable = GetPlayerTable(Player)

		PlayerTable.Tracer = Drawingnew("Line")

		PlayerTable.Connections.Tracer = RunService.RenderStepped:Connect(function()
			if Player.Character and Player.Character:FindFirstChildOfClass("Humanoid") and Player.Character:FindFirstChild("HumanoidRootPart") and Player.Character:FindFirstChild("Head") and Environment.Settings.Enabled then
				local HRPCFrame, HRPSize = Player.Character.HumanoidRootPart.CFrame, Player.Character.HumanoidRootPart.Size
				local _3DVector, OnScreen = WorldToViewportPoint(HRPCFrame * CFramenew(0, -HRPSize.Y - 0.5, 0).Position)
				local _2DVector = WorldToViewportPoint(Player.Character.HumanoidRootPart.Position)

				local HeadOffset = WorldToViewportPoint(Player.Character.Head.Position + Vector3new(0, 0.5, 0))
				local LegsOffset = WorldToViewportPoint(Player.Character.HumanoidRootPart.Position - Vector3new(0, 1.5, 0))

				if OnScreen and Environment.Visuals.TracersSettings.Enabled then
					if Environment.Visuals.TracersSettings.Enabled then
						PlayerTable.Tracer.Visible = PlayerTable.Checks.Alive and PlayerTable.Checks.Team and true or false

						if PlayerTable.Tracer.Visible then
							PlayerTable.Tracer.Thickness = Environment.Visuals.TracersSettings.Thickness
							PlayerTable.Tracer.Color = Environment.Visuals.TracersSettings.Color
							PlayerTable.Tracer.Transparency = Environment.Visuals.TracersSettings.Transparency

							PlayerTable.Tracer.To = Environment.Visuals.BoxSettings.Type == 1 and Vector2new(_3DVector.X, _3DVector.Y) or Vector2new(_2DVector.X, _2DVector.Y - (HeadOffset.Y - LegsOffset.Y) * 0.75)

							if Environment.Visuals.TracersSettings.Type == 1 then
								PlayerTable.Tracer.From = Vector2new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
							elseif Environment.Visuals.TracersSettings.Type == 2 then
								PlayerTable.Tracer.From = Vector2new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
							elseif Environment.Visuals.TracersSettings.Type == 3 then
								PlayerTable.Tracer.From = Vector2new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
							else
								PlayerTable.Tracer.From = Vector2new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
							end
						end
					end
				else
					PlayerTable.Tracer.Visible = false
				end
			else
				PlayerTable.Tracer.Visible = false
			end
		end)
	end,

	AddBox = function(Player)
		local PlayerTable = GetPlayerTable(Player)

		PlayerTable.Box.Square = Drawingnew("Square")

		PlayerTable.Box.TopLeftLine = Drawingnew("Line")
		PlayerTable.Box.TopLeftLine = Drawingnew("Line")
		PlayerTable.Box.TopRightLine = Drawingnew("Line")
		PlayerTable.Box.BottomLeftLine = Drawingnew("Line")
		PlayerTable.Box.BottomRightLine = Drawingnew("Line")

		local function Visibility(Bool)
			if Environment.Visuals.BoxSettings.Type == 1 then
				PlayerTable.Box.Square.Visible = not Bool

				PlayerTable.Box.TopLeftLine.Visible = Bool
				PlayerTable.Box.TopRightLine.Visible = Bool
				PlayerTable.Box.BottomLeftLine.Visible = Bool
				PlayerTable.Box.BottomRightLine.Visible = Bool
			elseif Environment.Visuals.BoxSettings.Type == 2 then
				PlayerTable.Box.Square.Visible = Bool

				PlayerTable.Box.TopLeftLine.Visible = not Bool
				PlayerTable.Box.TopRightLine.Visible = not Bool
				PlayerTable.Box.BottomLeftLine.Visible = not Bool
				PlayerTable.Box.BottomRightLine.Visible = not Bool
			end
		end

		local function Visibility2(Bool)
			PlayerTable.Box.Square.Visible = Bool

			PlayerTable.Box.TopLeftLine.Visible = Bool
			PlayerTable.Box.TopRightLine.Visible = Bool
			PlayerTable.Box.BottomLeftLine.Visible = Bool
			PlayerTable.Box.BottomRightLine.Visible = Bool
		end

		PlayerTable.Connections.Box = RunService.RenderStepped:Connect(function()
			if Player.Character and Player.Character:FindFirstChildOfClass("Humanoid") and Player.Character:FindFirstChild("HumanoidRootPart") and Player.Character:FindFirstChild("Head") and Environment.Settings.Enabled then
				local Vector, OnScreen = WorldToViewportPoint(Player.Character.HumanoidRootPart.Position)

				Visibility(Environment.Visuals.BoxSettings.Enabled)

				if OnScreen and Environment.Visuals.BoxSettings.Enabled then
					if PlayerTable.Checks.Alive and PlayerTable.Checks.Team then
						Visibility(true)
					else
						Visibility2(false)
					end

					local HRPCFrame, HRPSize = Player.Character.HumanoidRootPart.CFrame, Player.Character.HumanoidRootPart.Size * Environment.Visuals.BoxSettings.Increase

					local HeadOffset = WorldToViewportPoint(Player.Character.Head.Position + Vector3new(0, 0.5, 0))
					local LegsOffset = WorldToViewportPoint(Player.Character.HumanoidRootPart.Position - Vector3new(0, 3, 0))

					local TopLeftPosition = WorldToViewportPoint(HRPCFrame * CFramenew(HRPSize.X, HRPSize.Y, 0).Position)
					local TopRightPosition = WorldToViewportPoint(HRPCFrame * CFramenew(-HRPSize.X, HRPSize.Y, 0).Position)
					local BottomLeftPosition = WorldToViewportPoint(HRPCFrame * CFramenew(HRPSize.X, -HRPSize.Y - 0.5, 0).Position)
					local BottomRightPosition = WorldToViewportPoint(HRPCFrame * CFramenew(-HRPSize.X, -HRPSize.Y - 0.5, 0).Position)

					if PlayerTable.Box.Square.Visible and not PlayerTable.Box.TopLeftLine.Visible and not PlayerTable.Box.TopRightLine.Visible and not PlayerTable.Box.BottomLeftLine.Visible and not PlayerTable.Box.BottomRightLine.Visible then
						PlayerTable.Box.Square.Thickness = Environment.Visuals.BoxSettings.Thickness
						PlayerTable.Box.Square.Color = Environment.Visuals.BoxSettings.Color
						PlayerTable.Box.Square.Transparency = Environment.Visuals.BoxSettings.Transparency
						PlayerTable.Box.Square.Filled = Environment.Visuals.BoxSettings.Filled

						PlayerTable.Box.Square.Size = Vector2new(2000 / Vector.Z, HeadOffset.Y - LegsOffset.Y)
						PlayerTable.Box.Square.Position = Vector2new(Vector.X - PlayerTable.Box.Square.Size.X / 2, Vector.Y - PlayerTable.Box.Square.Size.Y / 2)
					elseif not PlayerTable.Box.Square.Visible and PlayerTable.Box.TopLeftLine.Visible and PlayerTable.Box.TopRightLine.Visible and PlayerTable.Box.BottomLeftLine.Visible and PlayerTable.Box.BottomRightLine.Visible then
						PlayerTable.Box.TopLeftLine.Thickness = Environment.Visuals.BoxSettings.Thickness
						PlayerTable.Box.TopLeftLine.Transparency = Environment.Visuals.BoxSettings.Transparency
						PlayerTable.Box.TopLeftLine.Color = Environment.Visuals.BoxSettings.Color

						PlayerTable.Box.TopRightLine.Thickness = Environment.Visuals.BoxSettings.Thickness
						PlayerTable.Box.TopRightLine.Transparency = Environment.Visuals.BoxSettings.Transparency
						PlayerTable.Box.TopRightLine.Color = Environment.Visuals.BoxSettings.Color

						PlayerTable.Box.BottomLeftLine.Thickness = Environment.Visuals.BoxSettings.Thickness
						PlayerTable.Box.BottomLeftLine.Transparency = Environment.Visuals.BoxSettings.Transparency
						PlayerTable.Box.BottomLeftLine.Color = Environment.Visuals.BoxSettings.Color

						PlayerTable.Box.BottomRightLine.Thickness = Environment.Visuals.BoxSettings.Thickness
						PlayerTable.Box.BottomRightLine.Transparency = Environment.Visuals.BoxSettings.Transparency
						PlayerTable.Box.BottomRightLine.Color = Environment.Visuals.BoxSettings.Color

						PlayerTable.Box.TopLeftLine.From = Vector2new(TopLeftPosition.X, TopLeftPosition.Y)
						PlayerTable.Box.TopLeftLine.To = Vector2new(TopRightPosition.X, TopRightPosition.Y)

						PlayerTable.Box.TopRightLine.From = Vector2new(TopRightPosition.X, TopRightPosition.Y)
						PlayerTable.Box.TopRightLine.To = Vector2new(BottomRightPosition.X, BottomRightPosition.Y)

						PlayerTable.Box.BottomLeftLine.From = Vector2new(BottomLeftPosition.X, BottomLeftPosition.Y)
						PlayerTable.Box.BottomLeftLine.To = Vector2new(TopLeftPosition.X, TopLeftPosition.Y)

						PlayerTable.Box.BottomRightLine.From = Vector2new(BottomRightPosition.X, BottomRightPosition.Y)
						PlayerTable.Box.BottomRightLine.To = Vector2new(BottomLeftPosition.X, BottomLeftPosition.Y)
					end
				else
					Visibility2(false)
				end
			else
				Visibility2(false)
			end
		end)
	end,

	AddHeadDot = function(Player)
		local PlayerTable = GetPlayerTable(Player)

		PlayerTable.HeadDot = Drawingnew("Circle")

		PlayerTable.Connections.HeadDot = RunService.RenderStepped:Connect(function()
			if Player.Character and Player.Character:FindFirstChildOfClass("Humanoid") and Player.Character:FindFirstChild("Head") and Environment.Settings.Enabled then
				local Vector, OnScreen = WorldToViewportPoint(Player.Character.Head.Position)

				PlayerTable.HeadDot.Visible = Environment.Visuals.HeadDotSettings.Enabled

				if OnScreen and Environment.Visuals.HeadDotSettings.Enabled then
					if Environment.Visuals.HeadDotSettings.Enabled then
						PlayerTable.HeadDot.Visible = PlayerTable.Checks.Alive and PlayerTable.Checks.Team and true or false

						if PlayerTable.HeadDot.Visible then
							PlayerTable.HeadDot.Thickness = Environment.Visuals.HeadDotSettings.Thickness
							PlayerTable.HeadDot.Color = Environment.Visuals.HeadDotSettings.Color
							PlayerTable.HeadDot.Transparency = Environment.Visuals.HeadDotSettings.Transparency
							PlayerTable.HeadDot.NumSides = Environment.Visuals.HeadDotSettings.Sides
							PlayerTable.HeadDot.Filled = Environment.Visuals.HeadDotSettings.Filled
							PlayerTable.HeadDot.Position = Vector2new(Vector.X, Vector.Y)

							local Top, Bottom = WorldToViewportPoint((Player.Character.Head.CFrame * CFramenew(0, Player.Character.Head.Size.Y / 2, 0)).Position), WorldToViewportPoint((Player.Character.Head.CFrame * CFramenew(0, -Player.Character.Head.Size.Y / 2, 0)).Position)
							PlayerTable.HeadDot.Radius = mathabs((Top - Bottom).Y) - 3
						end
					end
				else
					PlayerTable.HeadDot.Visible = false
				end
			else
				PlayerTable.HeadDot.Visible = false
			end
		end)
	end,

	AddHealthBar = function(Player)
		local PlayerTable = GetPlayerTable(Player)

		PlayerTable.HealthBar.Main = Drawingnew("Square")
		PlayerTable.HealthBar.Outline = Drawingnew("Square")

		PlayerTable.Connections.HealthBar = RunService.RenderStepped:Connect(function()
			if Player.Character and Player.Character:FindFirstChildOfClass("Humanoid") and Player.Character:FindFirstChild("HumanoidRootPart") and Environment.Settings.Enabled then
				local Vector, OnScreen = WorldToViewportPoint(Player.Character.HumanoidRootPart.Position)

				local LeftPosition = WorldToViewportPoint(Player.Character.HumanoidRootPart.CFrame * CFramenew(Player.Character.HumanoidRootPart.Size.X, Player.Character.HumanoidRootPart.Size.Y / 2, 0).Position)
				local RightPosition = WorldToViewportPoint(Player.Character.HumanoidRootPart.CFrame * CFramenew(-Player.Character.HumanoidRootPart.Size.X, Player.Character.HumanoidRootPart.Size.Y / 2, 0).Position)

				PlayerTable.HealthBar.Main.Visible = Environment.Visuals.HealthBarSettings.Enabled
				PlayerTable.HealthBar.Outline.Visible = Environment.Visuals.HealthBarSettings.Enabled

				if OnScreen and Environment.Visuals.HealthBarSettings.Enabled then
					if Environment.Visuals.HealthBarSettings.Enabled then
						local Humanoid = Player.Character:FindFirstChildOfClass("Humanoid")

						PlayerTable.HealthBar.Main.Visible = PlayerTable.Checks.Alive and PlayerTable.Checks.Team and true or false
						PlayerTable.HealthBar.Outline.Visible = PlayerTable.HealthBar.Main.Visible

						if PlayerTable.HealthBar.Main.Visible then
							PlayerTable.HealthBar.Main.Thickness = 1
							PlayerTable.HealthBar.Main.Color = Color3fromRGB(255 - mathfloor(Humanoid.Health / 100 * 255), mathfloor(Humanoid.Health / 100 * 255), Environment.Visuals.HealthBarSettings.Blue)
							PlayerTable.HealthBar.Main.Transparency = Environment.Visuals.HealthBarSettings.Transparency
							PlayerTable.HealthBar.Main.Filled = true
							PlayerTable.HealthBar.Main.ZIndex = 2

							PlayerTable.HealthBar.Outline.Thickness = 3
							PlayerTable.HealthBar.Outline.Color = Environment.Visuals.HealthBarSettings.OutlineColor
							PlayerTable.HealthBar.Outline.Transparency = Environment.Visuals.HealthBarSettings.Transparency
							PlayerTable.HealthBar.Outline.Filled = false
							PlayerTable.HealthBar.Main.ZIndex = 1

							if Environment.Visuals.HealthBarSettings.Type == 1 then
								PlayerTable.HealthBar.Outline.Size = Vector2new(2000 / Vector.Z, Environment.Visuals.HealthBarSettings.Size)
								PlayerTable.HealthBar.Main.Size = Vector2new(PlayerTable.HealthBar.Outline.Size.X * (Humanoid.Health / 100), PlayerTable.HealthBar.Outline.Size.Y)
								PlayerTable.HealthBar.Main.Position = Vector2new(Vector.X - PlayerTable.HealthBar.Outline.Size.X / 2, Vector.Y - PlayerTable.HealthBar.Outline.Size.X / 2 - Environment.Visuals.HealthBarSettings.Offset)
							elseif Environment.Visuals.HealthBarSettings.Type == 2 then
								PlayerTable.HealthBar.Outline.Size = Vector2new(2000 / Vector.Z, Environment.Visuals.HealthBarSettings.Size)
								PlayerTable.HealthBar.Main.Size = Vector2new(PlayerTable.HealthBar.Outline.Size.X * (Humanoid.Health / 100), PlayerTable.HealthBar.Outline.Size.Y)
								PlayerTable.HealthBar.Main.Position = Vector2new(Vector.X - PlayerTable.HealthBar.Outline.Size.X / 2, Vector.Y + PlayerTable.HealthBar.Outline.Size.X / 2 + Environment.Visuals.HealthBarSettings.Offset)
							elseif Environment.Visuals.HealthBarSettings.Type == 3 then
								PlayerTable.HealthBar.Outline.Size = Vector2new(Environment.Visuals.HealthBarSettings.Size, 2500 / Vector.Z)
								PlayerTable.HealthBar.Main.Size = Vector2new(PlayerTable.HealthBar.Outline.Size.X, PlayerTable.HealthBar.Outline.Size.Y * (Humanoid.Health / 100))
								PlayerTable.HealthBar.Main.Position = Vector2new(LeftPosition.X - Environment.Visuals.HealthBarSettings.Offset, Vector.Y - PlayerTable.HealthBar.Outline.Size.Y / 2)
							elseif Environment.Visuals.HealthBarSettings.Type == 4 then
								PlayerTable.HealthBar.Outline.Size = Vector2new(Environment.Visuals.HealthBarSettings.Size, 2500 / Vector.Z)
								PlayerTable.HealthBar.Main.Size = Vector2new(PlayerTable.HealthBar.Outline.Size.X, PlayerTable.HealthBar.Outline.Size.Y * (Humanoid.Health / 100))
								PlayerTable.HealthBar.Main.Position = Vector2new(RightPosition.X + Environment.Visuals.HealthBarSettings.Offset, Vector.Y - PlayerTable.HealthBar.Outline.Size.Y / 2)
							end

							PlayerTable.HealthBar.Outline.Position = PlayerTable.HealthBar.Main.Position
						end
					end
				else
					PlayerTable.HealthBar.Main.Visible = false
					PlayerTable.HealthBar.Outline.Visible = false
				end
			else
				PlayerTable.HealthBar.Main.Visible = false
				PlayerTable.HealthBar.Outline.Visible = false
			end
		end)
	end,

	AddCrosshair = function()
		local AxisX, AxisY = Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2

		ServiceConnections.AxisConnection = RunService.RenderStepped:Connect(function()
			if Environment.Crosshair.Settings.Enabled then
				if Environment.Crosshair.Settings.Type == 1 then
					AxisX, AxisY = UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y
				elseif Environment.Crosshair.Settings.Type == 2 then
					AxisX, AxisY = Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2
				else
					Environment.Crosshair.Settings.Type = 1
				end
			end
		end)

		ServiceConnections.CrosshairConnection = RunService.RenderStepped:Connect(function()
			if Environment.Crosshair.Settings.Enabled then

				--// Left Line

				Environment.Crosshair.Parts.LeftLine.Visible = Environment.Crosshair.Settings.Enabled
				Environment.Crosshair.Parts.LeftLine.Color = Environment.Crosshair.Settings.Color
				Environment.Crosshair.Parts.LeftLine.Thickness = Environment.Crosshair.Settings.Thickness
				Environment.Crosshair.Parts.LeftLine.Transparency = Environment.Crosshair.Settings.Transparency

				Environment.Crosshair.Parts.LeftLine.From = Vector2new(AxisX - (mathcos(mathrad(Environment.Crosshair.Settings.Rotation)) * Environment.Crosshair.Settings.GapSize), AxisY - (mathsin(mathrad(Environment.Crosshair.Settings.Rotation)) * Environment.Crosshair.Settings.GapSize))
				Environment.Crosshair.Parts.LeftLine.To = Vector2new(AxisX - (mathcos(mathrad(Environment.Crosshair.Settings.Rotation)) * (Environment.Crosshair.Settings.Size + Environment.Crosshair.Settings.GapSize)), AxisY - (mathsin(mathrad(Environment.Crosshair.Settings.Rotation)) * (Environment.Crosshair.Settings.Size + Environment.Crosshair.Settings.GapSize)))

				--// Right Line

				Environment.Crosshair.Parts.RightLine.Visible = Environment.Crosshair.Settings.Enabled
				Environment.Crosshair.Parts.RightLine.Color = Environment.Crosshair.Settings.Color
				Environment.Crosshair.Parts.RightLine.Thickness = Environment.Crosshair.Settings.Thickness
				Environment.Crosshair.Parts.RightLine.Transparency = Environment.Crosshair.Settings.Transparency


				Environment.Crosshair.Parts.RightLine.From = Vector2new(AxisX + (mathcos(mathrad(Environment.Crosshair.Settings.Rotation)) * Environment.Crosshair.Settings.GapSize), AxisY + (mathsin(mathrad(Environment.Crosshair.Settings.Rotation)) * Environment.Crosshair.Settings.GapSize))
				Environment.Crosshair.Parts.RightLine.To = Vector2new(AxisX + (mathcos(mathrad(Environment.Crosshair.Settings.Rotation)) * (Environment.Crosshair.Settings.Size + Environment.Crosshair.Settings.GapSize)), AxisY + (mathsin(mathrad(Environment.Crosshair.Settings.Rotation)) * (Environment.Crosshair.Settings.Size + Environment.Crosshair.Settings.GapSize)))

				--// Top Line

				Environment.Crosshair.Parts.TopLine.Visible = Environment.Crosshair.Settings.Enabled
				Environment.Crosshair.Parts.TopLine.Color = Environment.Crosshair.Settings.Color
				Environment.Crosshair.Parts.TopLine.Thickness = Environment.Crosshair.Settings.Thickness
				Environment.Crosshair.Parts.TopLine.Transparency = Environment.Crosshair.Settings.Transparency

				Environment.Crosshair.Parts.TopLine.From = Vector2new(AxisX - (mathsin(mathrad(-Environment.Crosshair.Settings.Rotation)) * Environment.Crosshair.Settings.GapSize), AxisY - (mathcos(mathrad(-Environment.Crosshair.Settings.Rotation)) * Environment.Crosshair.Settings.GapSize))
				Environment.Crosshair.Parts.TopLine.To = Vector2new(AxisX - (mathsin(mathrad(-Environment.Crosshair.Settings.Rotation)) * (Environment.Crosshair.Settings.Size + Environment.Crosshair.Settings.GapSize)), AxisY - (mathcos(mathrad(-Environment.Crosshair.Settings.Rotation)) * (Environment.Crosshair.Settings.Size + Environment.Crosshair.Settings.GapSize)))

				--// Bottom Line

				Environment.Crosshair.Parts.BottomLine.Visible = Environment.Crosshair.Settings.Enabled
				Environment.Crosshair.Parts.BottomLine.Color = Environment.Crosshair.Settings.Color
				Environment.Crosshair.Parts.BottomLine.Thickness = Environment.Crosshair.Settings.Thickness
				Environment.Crosshair.Parts.BottomLine.Transparency = Environment.Crosshair.Settings.Transparency

				Environment.Crosshair.Parts.BottomLine.From = Vector2new(AxisX + (mathsin(mathrad(-Environment.Crosshair.Settings.Rotation)) * Environment.Crosshair.Settings.GapSize), AxisY + (mathcos(mathrad(-Environment.Crosshair.Settings.Rotation)) * Environment.Crosshair.Settings.GapSize))
				Environment.Crosshair.Parts.BottomLine.To = Vector2new(AxisX + (mathsin(mathrad(-Environment.Crosshair.Settings.Rotation)) * (Environment.Crosshair.Settings.Size + Environment.Crosshair.Settings.GapSize)), AxisY + (mathcos(mathrad(-Environment.Crosshair.Settings.Rotation)) * (Environment.Crosshair.Settings.Size + Environment.Crosshair.Settings.GapSize)))

				--// Center Dot

				Environment.Crosshair.Parts.CenterDot.Visible = Environment.Crosshair.Settings.Enabled and Environment.Crosshair.Settings.CenterDot
				Environment.Crosshair.Parts.CenterDot.Color = Environment.Crosshair.Settings.CenterDotColor
				Environment.Crosshair.Parts.CenterDot.Radius = Environment.Crosshair.Settings.CenterDotSize
				Environment.Crosshair.Parts.CenterDot.Transparency = Environment.Crosshair.Settings.CenterDotTransparency
				Environment.Crosshair.Parts.CenterDot.Filled = Environment.Crosshair.Settings.CenterDotFilled
				Environment.Crosshair.Parts.CenterDot.Thickness = Environment.Crosshair.Settings.CenterDotThickness

				Environment.Crosshair.Parts.CenterDot.Position = Vector2new(AxisX, AxisY)
			else
				Environment.Crosshair.Parts.LeftLine.Visible = false
				Environment.Crosshair.Parts.RightLine.Visible = false
				Environment.Crosshair.Parts.TopLine.Visible = false
				Environment.Crosshair.Parts.BottomLine.Visible = false
				Environment.Crosshair.Parts.CenterDot.Visible = false
			end
		end)
	end
}

--// Functions

local function Wrap(Player)
	if not GetPlayerTable(Player) then
		local Table, Value = nil, {Name = Player.Name, Checks = {Alive = true, Team = true}, Connections = {}, ESP = nil, Tracer = nil, HeadDot = nil, HealthBar = {Main = nil, Outline = nil}, Box = {Square = nil, TopLeftLine = nil, TopRightLine = nil, BottomLeftLine = nil, BottomRightLine = nil}, Chams = {}}

		for _, v in next, Environment.WrappedPlayers do
			if v[1] == Player.Name then
				Table = v
			end
		end

		if not Table then
			Environment.WrappedPlayers[#Environment.WrappedPlayers + 1] = Value
			AssignRigType(Player)
			InitChecks(Player)

			Visuals.AddChams(Player)
			Visuals.AddESP(Player)
			Visuals.AddTracer(Player)
			Visuals.AddBox(Player)
			Visuals.AddHeadDot(Player)
			Visuals.AddHealthBar(Player)
		end
	end
end

local function UnWrap(Player)
	local Table, Index = nil, nil

	for i, v in next, Environment.WrappedPlayers do
		if v.Name == Player.Name then
			Table, Index = v, i
		end
	end

	if Table then
		for _, v in next, Table.Connections do
			v:Disconnect()
		end

		pcall(function()
			Table.ESP:Remove()
			Table.Tracer:Remove()
			Table.HeadDot:Remove()
			Table.HealthBar.Main:Remove()
			Table.HealthBar.Outline:Remove()
		end)

		for _, v in next, Table.Box do
			if not v.Remove then
				continue
			else
				v:Remove()
			end
		end

		for _, v in next, Table.Chams do
			for _, v2 in next, v do
				if not v2.Remove then
					continue
				else
					v2:Remove()
				end
			end
		end

		Environment.WrappedPlayers[Index] = nil
	end
end

local function Load()
	Visuals.AddCrosshair()

	ServiceConnections.PlayerAddedConnection = Players.PlayerAdded:Connect(Wrap)
	ServiceConnections.PlayerRemovingConnection = Players.PlayerRemoving:Connect(UnWrap)

	ServiceConnections.ReWrapPlayers = RunService.RenderStepped:Connect(function()
		for _, v in next, Players:GetPlayers() do
			if v ~= LocalPlayer then
				Wrap(v)
			end
		end

		wait(30)
	end)
end

--// Functions

Environment.Functions = {}

function Environment.Functions:Exit()
	for _, v in next, ServiceConnections do
		v:Disconnect()
	end

	for _, v in next, Environment.Crosshair.Parts do
		v:Remove()
	end

	for _, v in next, Players:GetPlayers() do
		if v ~= LocalPlayer then
			UnWrap(v)
		end
	end

	getgenv().AirHub.WallHack.Functions = nil
	getgenv().AirHub.WallHack = nil

	Load = nil; GetPlayerTable = nil; AssignRigType = nil; InitChecks = nil; UpdateCham = nil; Visuals = nil; Wrap = nil; UnWrap = nil
end

function Environment.Functions:Restart()
	for _, v in next, Players:GetPlayers() do
		if v ~= LocalPlayer then
			UnWrap(v)
		end
	end

	for _, v in next, ServiceConnections do
		v:Disconnect()
	end

	Load()
end

function Environment.Functions:ResetSettings()
	Environment.Visuals = {
		ChamsSettings = {
			Enabled = false,
			Color = Color3fromRGB(255, 255, 255),
			Transparency = 0.2,
			Thickness = 0,
			Filled = true,
			EntireBody = false -- For R15, keep false to prevent lag
		},

		ESPSettings = {
			Enabled = true,
			TextColor = Color3fromRGB(255, 255, 255),
			TextSize = 14,
			Center = true,
			Outline = true,
			OutlineColor = Color3fromRGB(0, 0, 0),
			TextTransparency = 0.7,
			TextFont = Drawing.Fonts.UI, -- UI, System, Plex, Monospace
			DisplayDistance = true,
			DisplayHealth = true,
			DisplayName = true
		},

		TracersSettings = {
			Enabled = true,
			Type = 1, -- 1 - Bottom; 2 - Center; 3 - Mouse
			Transparency = 0.7,
			Thickness = 1,
			Color = Color3fromRGB(255, 255, 255)
		},

		BoxSettings = {
			Enabled = true,
			Type = 1; -- 1 - 3D; 2 - 2D
			Color = Color3fromRGB(255, 255, 255),
			Transparency = 0.7,
			Thickness = 1,
			Filled = false, -- For 2D
			Increase = 1
		},

		HeadDotSettings = {
			Enabled = true,
			Color = Color3fromRGB(255, 255, 255),
			Transparency = 0.5,
			Thickness = 1,
			Filled = true,
			Sides = 30
		},

		HealthBarSettings = {
			Enabled = false,
			Transparency = 0.8,
			Size = 2,
			Offset = 10,
			OutlineColor = Color3fromRGB(0, 0, 0),
			Blue = 50,
			Type = 3 -- 1 - Top; 2 - Bottom; 3 - Left; 4 - Right
		}
	}

	Environment.Crosshair.Settings = {
		Enabled = false,
		Type = 1, -- 1 - Mouse; 2 - Center
		Size = 12,
		Thickness = 1,
		Color = Color3fromRGB(0, 255, 0),
		Transparency = 1,
		GapSize = 5,
		CenterDot = false,
		CenterDotColor = Color3fromRGB(0, 255, 0),
		CenterDotSize = 1,
		CenterDotTransparency = 1,
		CenterDotFilled = true,
		CenterDotThickness = 1
	}

	Environment.Settings = {
		Enabled = false,
		TeamCheck = false,
		AliveCheck = true
	}
end

setmetatable(Environment.Functions, {
	__newindex = warn
})

--// Main

Load()
--[[

	AirHub by Exunys © CC0 1.0 Universal (2023)

	https://github.com/Exunys

]]

--// Cache

local loadstring, getgenv, setclipboard, tablefind, UserInputService = loadstring, getgenv, setclipboard, table.find, game:GetService("UserInputService")

--// Loaded check

if AirHub or AirHubV2Loaded then
    return
end

--// Environment

getgenv().AirHub = {}

--// Load Modules

loadstring(game:HttpGet("https://raw.githubusercontent.com/Exunys/AirHub/main/Modules/Aimbot.lua"))()
loadstring(game:HttpGet("https://raw.githubusercontent.com/Exunys/AirHub/main/Modules/Wall%20Hack.lua"))()

--// Variables

local Library = loadstring(game:GetObjects("rbxassetid://7657867786")[1].Source)() -- Pepsi's UI Library
local Aimbot, WallHack = getgenv().AirHub.Aimbot, getgenv().AirHub.WallHack
local Parts, Fonts, TracersType = {"Head", "HumanoidRootPart", "Torso", "Left Arm", "Right Arm", "Left Leg", "Right Leg", "LeftHand", "RightHand", "LeftLowerArm", "RightLowerArm", "LeftUpperArm", "RightUpperArm", "LeftFoot", "LeftLowerLeg", "UpperTorso", "LeftUpperLeg", "RightFoot", "RightLowerLeg", "LowerTorso", "RightUpperLeg"}, {"UI", "System", "Plex", "Monospace"}, {"Bottom", "Center", "Mouse"}

--// Frame

Library.UnloadCallback = function()
	Aimbot.Functions:Exit()
	WallHack.Functions:Exit()
	getgenv().AirHub = nil
end

local MainFrame = Library:CreateWindow({
	Name = "AirHub",
	Themeable = {
		Image = "7059346386",
		Info = "Made by Exunys\nPowered by Pepsi's UI Library",
		Credit = false
	},
	Background = "",
	Theme = [[{"__Designer.Colors.topGradient":"3F0C64","__Designer.Colors.section":"C259FB","__Designer.Colors.hoveredOptionBottom":"4819B4","__Designer.Background.ImageAssetID":"rbxassetid://4427304036","__Designer.Colors.selectedOption":"4E149C","__Designer.Colors.unselectedOption":"482271","__Designer.Files.WorkspaceFile":"AirHub","__Designer.Colors.unhoveredOptionTop":"310269","__Designer.Colors.outerBorder":"391D57","__Designer.Background.ImageColor":"69009C","__Designer.Colors.tabText":"B9B9B9","__Designer.Colors.elementBorder":"160B24","__Designer.Background.ImageTransparency":100,"__Designer.Colors.background":"1E1237","__Designer.Colors.innerBorder":"531E79","__Designer.Colors.bottomGradient":"361A60","__Designer.Colors.sectionBackground":"21002C","__Designer.Colors.hoveredOptionTop":"6B10F9","__Designer.Colors.otherElementText":"7B44A8","__Designer.Colors.main":"AB26FF","__Designer.Colors.elementText":"9F7DB5","__Designer.Colors.unhoveredOptionBottom":"3E0088","__Designer.Background.UseBackgroundImage":false}]]
})

--// Tabs

local AimbotTab = MainFrame:CreateTab({
	Name = "Aimbot"
})

local VisualsTab = MainFrame:CreateTab({
	Name = "Visuals"
})

local CrosshairTab = MainFrame:CreateTab({
	Name = "Crosshair"
})

local FunctionsTab = MainFrame:CreateTab({
	Name = "Functions"
})

--// Aimbot Sections

local Values = AimbotTab:CreateSection({
	Name = "Values"
})

local Checks = AimbotTab:CreateSection({
	Name = "Checks"
})

local ThirdPerson = AimbotTab:CreateSection({
	Name = "Third Person"
})

local FOV_Values = AimbotTab:CreateSection({
	Name = "Field Of View",
	Side = "Right"
})

local FOV_Appearance = AimbotTab:CreateSection({
	Name = "FOV Circle Appearance",
	Side = "Right"
})

--// Visuals Sections

local WallHackChecks = VisualsTab:CreateSection({
	Name = "Checks"
})

local ESPSettings = VisualsTab:CreateSection({
	Name = "ESP Settings"
})

local BoxesSettings = VisualsTab:CreateSection({
	Name = "Boxes Settings"
})

local ChamsSettings = VisualsTab:CreateSection({
	Name = "Chams Settings"
})

local TracersSettings = VisualsTab:CreateSection({
	Name = "Tracers Settings",
	Side = "Right"
})

local HeadDotsSettings = VisualsTab:CreateSection({
	Name = "Head Dots Settings",
	Side = "Right"
})

local HealthBarSettings = VisualsTab:CreateSection({
	Name = "Health Bar Settings",
	Side = "Right"
})

--// Crosshair Sections

local CrosshairSettings = CrosshairTab:CreateSection({
	Name = "Settings"
})

local CrosshairSettings_CenterDot = CrosshairTab:CreateSection({
	Name = "Center Dot Settings",
	Side = "Right"
})

--// Functions Sections

local FunctionsSection = FunctionsTab:CreateSection({
	Name = "Functions"
})

--// Aimbot Values

Values:AddToggle({
	Name = "Enabled",
	Value = Aimbot.Settings.Enabled,
	Callback = function(New, Old)
		Aimbot.Settings.Enabled = New
	end
}).Default = Aimbot.Settings.Enabled

Values:AddToggle({
	Name = "Toggle",
	Value = Aimbot.Settings.Toggle,
	Callback = function(New, Old)
		Aimbot.Settings.Toggle = New
	end
}).Default = Aimbot.Settings.Toggle

Aimbot.Settings.LockPart = Parts[1]; Values:AddDropdown({
	Name = "Lock Part",
	Value = Parts[1],
	Callback = function(New, Old)
		Aimbot.Settings.LockPart = New
	end,
	List = Parts,
	Nothing = "Head"
}).Default = Parts[1]

Values:AddTextbox({ -- Using a Textbox instead of a Keybind because the UI Library doesn't support Mouse inputs like Left Click / Right Click...
	Name = "Hotkey",
	Value = Aimbot.Settings.TriggerKey,
	Callback = function(New, Old)
		Aimbot.Settings.TriggerKey = New
	end
}).Default = Aimbot.Settings.TriggerKey

--[[
Values:AddKeybind({
	Name = "Hotkey",
	Value = Aimbot.Settings.TriggerKey,
	Callback = function(New, Old)
		Aimbot.Settings.TriggerKey = stringmatch(tostring(New), "Enum%.[UserInputType]*[KeyCode]*%.(.+)")
	end,
}).Default = Aimbot.Settings.TriggerKey
]]

Values:AddSlider({
	Name = "Sensitivity",
	Value = Aimbot.Settings.Sensitivity,
	Callback = function(New, Old)
		Aimbot.Settings.Sensitivity = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = Aimbot.Settings.Sensitivity

--// Aimbot Checks

Checks:AddToggle({
	Name = "Team Check",
	Value = Aimbot.Settings.TeamCheck,
	Callback = function(New, Old)
		Aimbot.Settings.TeamCheck = New
	end
}).Default = Aimbot.Settings.TeamCheck

Checks:AddToggle({
	Name = "Wall Check",
	Value = Aimbot.Settings.WallCheck,
	Callback = function(New, Old)
		Aimbot.Settings.WallCheck = New
	end
}).Default = Aimbot.Settings.WallCheck

Checks:AddToggle({
	Name = "Alive Check",
	Value = Aimbot.Settings.AliveCheck,
	Callback = function(New, Old)
		Aimbot.Settings.AliveCheck = New
	end
}).Default = Aimbot.Settings.AliveCheck

--// Aimbot ThirdPerson

ThirdPerson:AddToggle({
	Name = "Enable Third Person",
	Value = Aimbot.Settings.ThirdPerson,
	Callback = function(New, Old)
		Aimbot.Settings.ThirdPerson = New
	end
}).Default = Aimbot.Settings.ThirdPerson

ThirdPerson:AddSlider({
	Name = "Sensitivity",
	Value = Aimbot.Settings.ThirdPersonSensitivity,
	Callback = function(New, Old)
		Aimbot.Settings.ThirdPersonSensitivity = New
	end,
	Min = 0.1,
	Max = 5,
	Decimals = 1
}).Default = Aimbot.Settings.ThirdPersonSensitivity

--// FOV Settings Values

FOV_Values:AddToggle({
	Name = "Enabled",
	Value = Aimbot.FOVSettings.Enabled,
	Callback = function(New, Old)
		Aimbot.FOVSettings.Enabled = New
	end
}).Default = Aimbot.FOVSettings.Enabled

FOV_Values:AddToggle({
	Name = "Visible",
	Value = Aimbot.FOVSettings.Visible,
	Callback = function(New, Old)
		Aimbot.FOVSettings.Visible = New
	end
}).Default = Aimbot.FOVSettings.Visible

FOV_Values:AddSlider({
	Name = "Amount",
	Value = Aimbot.FOVSettings.Amount,
	Callback = function(New, Old)
		Aimbot.FOVSettings.Amount = New
	end,
	Min = 10,
	Max = 300
}).Default = Aimbot.FOVSettings.Amount

--// FOV Settings Appearance

FOV_Appearance:AddToggle({
	Name = "Filled",
	Value = Aimbot.FOVSettings.Filled,
	Callback = function(New, Old)
		Aimbot.FOVSettings.Filled = New
	end
}).Default = Aimbot.FOVSettings.Filled

FOV_Appearance:AddSlider({
	Name = "Transparency",
	Value = Aimbot.FOVSettings.Transparency,
	Callback = function(New, Old)
		Aimbot.FOVSettings.Transparency = New
	end,
	Min = 0,
	Max = 1,
	Decimal = 1
}).Default = Aimbot.FOVSettings.Transparency

FOV_Appearance:AddSlider({
	Name = "Sides",
	Value = Aimbot.FOVSettings.Sides,
	Callback = function(New, Old)
		Aimbot.FOVSettings.Sides = New
	end,
	Min = 3,
	Max = 60
}).Default = Aimbot.FOVSettings.Sides

FOV_Appearance:AddSlider({
	Name = "Thickness",
	Value = Aimbot.FOVSettings.Thickness,
	Callback = function(New, Old)
		Aimbot.FOVSettings.Thickness = New
	end,
	Min = 1,
	Max = 50
}).Default = Aimbot.FOVSettings.Thickness

FOV_Appearance:AddColorpicker({
	Name = "Color",
	Value = Aimbot.FOVSettings.Color,
	Callback = function(New, Old)
		Aimbot.FOVSettings.Color = New
	end
}).Default = Aimbot.FOVSettings.Color

FOV_Appearance:AddColorpicker({
	Name = "Locked Color",
	Value = Aimbot.FOVSettings.LockedColor,
	Callback = function(New, Old)
		Aimbot.FOVSettings.LockedColor = New
	end
}).Default = Aimbot.FOVSettings.LockedColor

--// Wall Hack Settings

WallHackChecks:AddToggle({
	Name = "Enabled",
	Value = WallHack.Settings.Enabled,
	Callback = function(New, Old)
		WallHack.Settings.Enabled = New
	end
}).Default = WallHack.Settings.Enabled

WallHackChecks:AddToggle({
	Name = "Team Check",
	Value = WallHack.Settings.TeamCheck,
	Callback = function(New, Old)
		WallHack.Settings.TeamCheck = New
	end
}).Default = WallHack.Settings.TeamCheck

WallHackChecks:AddToggle({
	Name = "Alive Check",
	Value = WallHack.Settings.AliveCheck,
	Callback = function(New, Old)
		WallHack.Settings.AliveCheck = New
	end
}).Default = WallHack.Settings.AliveCheck

--// Visuals Settings

ESPSettings:AddToggle({
	Name = "Enabled",
	Value = WallHack.Visuals.ESPSettings.Enabled,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.Enabled = New
	end
}).Default = WallHack.Visuals.ESPSettings.Enabled

ESPSettings:AddToggle({
	Name = "Outline",
	Value = WallHack.Visuals.ESPSettings.Outline,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.Outline = New
	end
}).Default = WallHack.Visuals.ESPSettings.Outline

ESPSettings:AddToggle({
	Name = "Display Distance",
	Value = WallHack.Visuals.ESPSettings.DisplayDistance,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.DisplayDistance = New
	end
}).Default = WallHack.Visuals.ESPSettings.DisplayDistance

ESPSettings:AddToggle({
	Name = "Display Health",
	Value = WallHack.Visuals.ESPSettings.DisplayHealth,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.DisplayHealth = New
	end
}).Default = WallHack.Visuals.ESPSettings.DisplayHealth

ESPSettings:AddToggle({
	Name = "Display Name",
	Value = WallHack.Visuals.ESPSettings.DisplayName,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.DisplayName = New
	end
}).Default = WallHack.Visuals.ESPSettings.DisplayName

ESPSettings:AddSlider({
	Name = "Offset",
	Value = WallHack.Visuals.ESPSettings.Offset,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.Offset = New
	end,
	Min = -30,
	Max = 30
}).Default = WallHack.Visuals.ESPSettings.Offset

ESPSettings:AddColorpicker({
	Name = "Text Color",
	Value = WallHack.Visuals.ESPSettings.TextColor,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.TextColor = New
	end
}).Default = WallHack.Visuals.ESPSettings.TextColor

ESPSettings:AddColorpicker({
	Name = "Outline Color",
	Value = WallHack.Visuals.ESPSettings.OutlineColor,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.OutlineColor = New
	end
}).Default = WallHack.Visuals.ESPSettings.OutlineColor

ESPSettings:AddSlider({
	Name = "Text Transparency",
	Value = WallHack.Visuals.ESPSettings.TextTransparency,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.TextTransparency = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = WallHack.Visuals.ESPSettings.TextTransparency

ESPSettings:AddSlider({
	Name = "Text Size",
	Value = WallHack.Visuals.ESPSettings.TextSize,
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.TextSize = New
	end,
	Min = 8,
	Max = 24
}).Default = WallHack.Visuals.ESPSettings.TextSize

ESPSettings:AddDropdown({
	Name = "Text Font",
	Value = Fonts[WallHack.Visuals.ESPSettings.TextFont + 1],
	Callback = function(New, Old)
		WallHack.Visuals.ESPSettings.TextFont = Drawing.Fonts[New]
	end,
	List = Fonts,
	Nothing = "UI"
}).Default = Fonts[WallHack.Visuals.ESPSettings.TextFont + 1]

BoxesSettings:AddToggle({
	Name = "Enabled",
	Value = WallHack.Visuals.BoxSettings.Enabled,
	Callback = function(New, Old)
		WallHack.Visuals.BoxSettings.Enabled = New
	end
}).Default = WallHack.Visuals.BoxSettings.Enabled

BoxesSettings:AddSlider({
	Name = "Transparency",
	Value = WallHack.Visuals.BoxSettings.Transparency,
	Callback = function(New, Old)
		WallHack.Visuals.BoxSettings.Transparency = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = WallHack.Visuals.BoxSettings.Transparency

BoxesSettings:AddSlider({
	Name = "Thickness",
	Value = WallHack.Visuals.BoxSettings.Thickness,
	Callback = function(New, Old)
		WallHack.Visuals.BoxSettings.Thickness = New
	end,
	Min = 1,
	Max = 5
}).Default = WallHack.Visuals.BoxSettings.Thickness

BoxesSettings:AddSlider({
	Name = "Scale Increase (For 3D)",
	Value = WallHack.Visuals.BoxSettings.Increase,
	Callback = function(New, Old)
		WallHack.Visuals.BoxSettings.Increase = New
	end,
	Min = 1,
	Max = 5
}).Default = WallHack.Visuals.BoxSettings.Increase

BoxesSettings:AddColorpicker({
	Name = "Color",
	Value = WallHack.Visuals.BoxSettings.Color,
	Callback = function(New, Old)
		WallHack.Visuals.BoxSettings.Color = New
	end
}).Default = WallHack.Visuals.BoxSettings.Color

BoxesSettings:AddDropdown({
	Name = "Type",
	Value = WallHack.Visuals.BoxSettings.Type == 1 and "3D" or "2D",
	Callback = function(New, Old)
		WallHack.Visuals.BoxSettings.Type = New == "3D" and 1 or 2
	end,
	List = {"3D", "2D"},
	Nothing = "3D"
}).Default = WallHack.Visuals.BoxSettings.Type == 1 and "3D" or "2D"

BoxesSettings:AddToggle({
	Name = "Filled (2D Square)",
	Value = WallHack.Visuals.BoxSettings.Filled,
	Callback = function(New, Old)
		WallHack.Visuals.BoxSettings.Filled = New
	end
}).Default = WallHack.Visuals.BoxSettings.Filled

ChamsSettings:AddToggle({
	Name = "Enabled",
	Value = WallHack.Visuals.ChamsSettings.Enabled,
	Callback = function(New, Old)
		WallHack.Visuals.ChamsSettings.Enabled = New
	end
}).Default = WallHack.Visuals.ChamsSettings.Enabled

ChamsSettings:AddToggle({
	Name = "Filled",
	Value = WallHack.Visuals.ChamsSettings.Filled,
	Callback = function(New, Old)
		WallHack.Visuals.ChamsSettings.Filled = New
	end
}).Default = WallHack.Visuals.ChamsSettings.Filled

ChamsSettings:AddToggle({
	Name = "Entire Body (For R15 Rigs)",
	Value = WallHack.Visuals.ChamsSettings.EntireBody,
	Callback = function(New, Old)
		WallHack.Visuals.ChamsSettings.EntireBody = New
	end
}).Default = WallHack.Visuals.ChamsSettings.EntireBody

ChamsSettings:AddSlider({
	Name = "Transparency",
	Value = WallHack.Visuals.ChamsSettings.Transparency,
	Callback = function(New, Old)
		WallHack.Visuals.ChamsSettings.Transparency = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = WallHack.Visuals.ChamsSettings.Transparency

ChamsSettings:AddSlider({
	Name = "Thickness",
	Value = WallHack.Visuals.ChamsSettings.Thickness,
	Callback = function(New, Old)
		WallHack.Visuals.ChamsSettings.Thickness = New
	end,
	Min = 0,
	Max = 3
}).Default = WallHack.Visuals.ChamsSettings.Thickness

ChamsSettings:AddColorpicker({
	Name = "Color",
	Value = WallHack.Visuals.ChamsSettings.Color,
	Callback = function(New, Old)
		WallHack.Visuals.ChamsSettings.Color = New
	end
}).Default = WallHack.Visuals.ChamsSettings.Color

TracersSettings:AddToggle({
	Name = "Enabled",
	Value = WallHack.Visuals.TracersSettings.Enabled,
	Callback = function(New, Old)
		WallHack.Visuals.TracersSettings.Enabled = New
	end
}).Default = WallHack.Visuals.TracersSettings.Enabled

TracersSettings:AddSlider({
	Name = "Transparency",
	Value = WallHack.Visuals.TracersSettings.Transparency,
	Callback = function(New, Old)
		WallHack.Visuals.TracersSettings.Transparency = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = WallHack.Visuals.TracersSettings.Transparency

TracersSettings:AddSlider({
	Name = "Thickness",
	Value = WallHack.Visuals.TracersSettings.Thickness,
	Callback = function(New, Old)
		WallHack.Visuals.TracersSettings.Thickness = New
	end,
	Min = 1,
	Max = 5
}).Default = WallHack.Visuals.TracersSettings.Thickness

TracersSettings:AddColorpicker({
	Name = "Color",
	Value = WallHack.Visuals.TracersSettings.Color,
	Callback = function(New, Old)
		WallHack.Visuals.TracersSettings.Color = New
	end
}).Default = WallHack.Visuals.TracersSettings.Color

TracersSettings:AddDropdown({
	Name = "Start From",
	Value = TracersType[WallHack.Visuals.TracersSettings.Type],
	Callback = function(New, Old)
		WallHack.Visuals.TracersSettings.Type = tablefind(TracersType, New)
	end,
	List = TracersType,
	Nothing = "Bottom"
}).Default = Fonts[WallHack.Visuals.TracersSettings.Type + 1]

HeadDotsSettings:AddToggle({
	Name = "Enabled",
	Value = WallHack.Visuals.HeadDotSettings.Enabled,
	Callback = function(New, Old)
		WallHack.Visuals.HeadDotSettings.Enabled = New
	end
}).Default = WallHack.Visuals.HeadDotSettings.Enabled

HeadDotsSettings:AddToggle({
	Name = "Filled",
	Value = WallHack.Visuals.HeadDotSettings.Filled,
	Callback = function(New, Old)
		WallHack.Visuals.HeadDotSettings.Filled = New
	end
}).Default = WallHack.Visuals.HeadDotSettings.Filled

HeadDotsSettings:AddSlider({
	Name = "Transparency",
	Value = WallHack.Visuals.HeadDotSettings.Transparency,
	Callback = function(New, Old)
		WallHack.Visuals.HeadDotSettings.Transparency = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = WallHack.Visuals.HeadDotSettings.Transparency

HeadDotsSettings:AddSlider({
	Name = "Thickness",
	Value = WallHack.Visuals.HeadDotSettings.Thickness,
	Callback = function(New, Old)
		WallHack.Visuals.HeadDotSettings.Thickness = New
	end,
	Min = 1,
	Max = 5
}).Default = WallHack.Visuals.HeadDotSettings.Thickness

HeadDotsSettings:AddSlider({
	Name = "Sides",
	Value = WallHack.Visuals.HeadDotSettings.Sides,
	Callback = function(New, Old)
		WallHack.Visuals.HeadDotSettings.Sides = New
	end,
	Min = 3,
	Max = 60
}).Default = WallHack.Visuals.HeadDotSettings.Sides

HeadDotsSettings:AddColorpicker({
	Name = "Color",
	Value = WallHack.Visuals.HeadDotSettings.Color,
	Callback = function(New, Old)
		WallHack.Visuals.HeadDotSettings.Color = New
	end
}).Default = WallHack.Visuals.HeadDotSettings.Color

HealthBarSettings:AddToggle({
	Name = "Enabled",
	Value = WallHack.Visuals.HealthBarSettings.Enabled,
	Callback = function(New, Old)
		WallHack.Visuals.HealthBarSettings.Enabled = New
	end
}).Default = WallHack.Visuals.HealthBarSettings.Enabled

HealthBarSettings:AddDropdown({
	Name = "Position",
	Value = WallHack.Visuals.HealthBarSettings.Type == 1 and "Top" or WallHack.Visuals.HealthBarSettings.Type == 2 and "Bottom" or WallHack.Visuals.HealthBarSettings.Type == 3 and "Left" or "Right",
	Callback = function(New, Old)
		WallHack.Visuals.HealthBarSettings.Type = New == "Top" and 1 or New == "Bottom" and 2 or New == "Left" and 3 or 4
	end,
	List = {"Top", "Bottom", "Left", "Right"},
	Nothing = "Left"
}).Default = WallHack.Visuals.HealthBarSettings.Type == 1 and "Top" or WallHack.Visuals.HealthBarSettings.Type == 2 and "Bottom" or WallHack.Visuals.HealthBarSettings.Type == 3 and "Left" or "Right"

HealthBarSettings:AddSlider({
	Name = "Transparency",
	Value = WallHack.Visuals.HealthBarSettings.Transparency,
	Callback = function(New, Old)
		WallHack.Visuals.HealthBarSettings.Transparency = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = WallHack.Visuals.HealthBarSettings.Transparency

HealthBarSettings:AddSlider({
	Name = "Size",
	Value = WallHack.Visuals.HealthBarSettings.Size,
	Callback = function(New, Old)
		WallHack.Visuals.HealthBarSettings.Size = New
	end,
	Min = 2,
	Max = 10
}).Default = WallHack.Visuals.HealthBarSettings.Size

HealthBarSettings:AddSlider({
	Name = "Blue",
	Value = WallHack.Visuals.HealthBarSettings.Blue,
	Callback = function(New, Old)
		WallHack.Visuals.HealthBarSettings.Blue = New
	end,
	Min = 0,
	Max = 255
}).Default = WallHack.Visuals.HealthBarSettings.Blue

HealthBarSettings:AddSlider({
	Name = "Offset",
	Value = WallHack.Visuals.HealthBarSettings.Offset,
	Callback = function(New, Old)
		WallHack.Visuals.HealthBarSettings.Offset = New
	end,
	Min = -30,
	Max = 30
}).Default = WallHack.Visuals.HealthBarSettings.Offset

HealthBarSettings:AddColorpicker({
	Name = "Outline Color",
	Value = WallHack.Visuals.HealthBarSettings.OutlineColor,
	Callback = function(New, Old)
		WallHack.Visuals.HealthBarSettings.OutlineColor = New
	end
}).Default = WallHack.Visuals.HealthBarSettings.OutlineColor

--// Crosshair Settings

CrosshairSettings:AddToggle({
	Name = "Mouse Cursor",
	Value = UserInputService.MouseIconEnabled,
	Callback = function(New, Old)
		UserInputService.MouseIconEnabled = New
	end
}).Default = UserInputService.MouseIconEnabled

CrosshairSettings:AddToggle({
	Name = "Enabled",
	Value = WallHack.Crosshair.Settings.Enabled,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.Enabled = New
	end
}).Default = WallHack.Crosshair.Settings.Enabled

CrosshairSettings:AddColorpicker({
	Name = "Color",
	Value = WallHack.Crosshair.Settings.Color,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.Color = New
	end
}).Default = WallHack.Crosshair.Settings.Color

CrosshairSettings:AddSlider({
	Name = "Transparency",
	Value = WallHack.Crosshair.Settings.Transparency,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.Transparency = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = WallHack.Crosshair.Settings.Transparency

CrosshairSettings:AddSlider({
	Name = "Size",
	Value = WallHack.Crosshair.Settings.Size,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.Size = New
	end,
	Min = 8,
	Max = 24
}).Default = WallHack.Crosshair.Settings.Size

CrosshairSettings:AddSlider({
	Name = "Thickness",
	Value = WallHack.Crosshair.Settings.Thickness,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.Thickness = New
	end,
	Min = 1,
	Max = 5
}).Default = WallHack.Crosshair.Settings.Thickness

CrosshairSettings:AddSlider({
	Name = "Gap Size",
	Value = WallHack.Crosshair.Settings.GapSize,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.GapSize = New
	end,
	Min = 0,
	Max = 20
}).Default = WallHack.Crosshair.Settings.GapSize

CrosshairSettings:AddSlider({
	Name = "Rotation (Degrees)",
	Value = WallHack.Crosshair.Settings.Rotation,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.Rotation = New
	end,
	Min = -180,
	Max = 180
}).Default = WallHack.Crosshair.Settings.Rotation

CrosshairSettings:AddDropdown({
	Name = "Position",
	Value = WallHack.Crosshair.Settings.Type == 1 and "Mouse" or "Center",
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.Type = New == "Mouse" and 1 or 2
	end,
	List = {"Mouse", "Center"},
	Nothing = "Mouse"
}).Default = WallHack.Crosshair.Settings.Type == 1 and "Mouse" or "Center"

CrosshairSettings_CenterDot:AddToggle({
	Name = "Center Dot",
	Value = WallHack.Crosshair.Settings.CenterDot,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.CenterDot = New
	end
}).Default = WallHack.Crosshair.Settings.CenterDot

CrosshairSettings_CenterDot:AddColorpicker({
	Name = "Center Dot Color",
	Value = WallHack.Crosshair.Settings.CenterDotColor,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.CenterDotColor = New
	end
}).Default = WallHack.Crosshair.Settings.CenterDotColor

CrosshairSettings_CenterDot:AddSlider({
	Name = "Center Dot Size",
	Value = WallHack.Crosshair.Settings.CenterDotSize,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.CenterDotSize = New
	end,
	Min = 1,
	Max = 6
}).Default = WallHack.Crosshair.Settings.CenterDotSize

CrosshairSettings_CenterDot:AddSlider({
	Name = "Center Dot Transparency",
	Value = WallHack.Crosshair.Settings.CenterDotTransparency,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.CenterDotTransparency = New
	end,
	Min = 0,
	Max = 1,
	Decimals = 2
}).Default = WallHack.Crosshair.Settings.CenterDotTransparency

CrosshairSettings_CenterDot:AddToggle({
	Name = "Center Dot Filled",
	Value = WallHack.Crosshair.Settings.CenterDotFilled,
	Callback = function(New, Old)
		WallHack.Crosshair.Settings.CenterDotFilled = New
	end
}).Default = WallHack.Crosshair.Settings.CenterDotFilled

--// Functions / Functions

FunctionsSection:AddButton({
	Name = "Reset Settings",
	Callback = function()
		Aimbot.Functions:ResetSettings()
		WallHack.Functions:ResetSettings()
		Library.ResetAll()
	end
})

FunctionsSection:AddButton({
	Name = "Restart",
	Callback = function()
		Aimbot.Functions:Restart()
		WallHack.Functions:Restart()
	end
})

FunctionsSection:AddButton({
	Name = "Exit",
	Callback = Library.Unload,
})

FunctionsSection:AddButton({
	Name = "Copy Script Page",
	Callback = function()
		setclipboard("https://github.com/Exunys/AirHub")
	end
})

--// AirHub V2 Prompt

do
	local Aux = Instance.new("BindableFunction")
    
	Aux.OnInvoke = function(Answer)
		if Answer == "No" then
			return
		end

		Library.Unload()
		loadstring(game:HttpGet("https://raw.githubusercontent.com/Exunys/AirHub-V2/main/src/Main.lua"))()
	end

	game.StarterGui:SetCore("SendNotification", {
		Title = "🎆  AirHub V2  🎆",
		Text = "Would you like to use the new AirHub V2 script?",
		Button1 = "Yes",
		Button2 = "No",
		Duration = 1 / 0,
		Icon = "rbxassetid://6238537240",
		Callback = Aux
	})
end
