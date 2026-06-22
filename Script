--[[
	================================================================================
	 DEVELOPER ADMIN PANEL  —  MADE BY RONTHEBARON
	 ESP DRAWING LAYER    —  PORTED FROM BLISSFUL4992/ESPs
	================================================================================
	Place in StarterPlayerScripts or run via executor while testing.
	Builds its own ScreenGui at runtime. 100 % client-side.

	TABS
	----
	  Movement   -> Fly, Noclip, InfJump, Sprint, Dash, DoubleJump, WallClimb,
	                Freeze, Swim-in-Air, WalkSpeed, JumpPower, Gravity, CharScale,
	                Freecam, FOV, SavedLocations, CinematicCam, CamPaths,
	                CamShake, SlowMo, TPDistance, FPLock, OrbitCam, ScreenshotMode,
	                DepthOfField, Respawn, UndoTeleport
	  Visuals    -> ESP (Box/Corner-Box/Tracer/Name/Dist/HealthBar/Skeleton/Team/Item/Vehicle/Objective),
	                Fullbright, InvisibleChar, GodModeVisual, RainbowChar,
	                TrailCreator, AuraEffect, FloatingText, HUD Overlay
	  World      -> TimeOfDay, Weather, Fog, Brightness, ColorCorrection,
	                XRay, HighlightInteractables, RemoveClutter, Explosion/Lightning/
	                Meteor/Earthquake effects, Particles, Soundboard
	  Players    -> Player list, Teleport, Spectate
	  Developer  -> RemoteEvent/Function logger, Instance explorer, Property editor,
	                Script perf, Memory, Network stats, Part count,
	                NPC tracker, Error log, Collision/Hitbox/Pathfinding vis,
	                Lag/FPS/Ping simulators
	  QoL        -> Favorites, Recents, Profiles, Theme editor, Notifications history
	  Keybinds   -> All feature keybinds + panel toggle
	  Settings   -> Theme, Notifications, Data management
	================================================================================
	CHANGELOG (Blissful ESP integration)
	--------------------------------------
	REPLACED: Drawing.new("Square") box → Drawing.new("Quad") with a matching black
	          border Quad behind it, matching Blissful's 2D Box ESP exactly.
	REPLACED: Off-screen Triangle arrow tracers → Drawing.new("Line") tracer from the
	          bottom-centre of the screen to the target's feet, plus a black shadow
	          line behind it — matching Blissful's tracer style precisely.
	ADDED:    Health Bars — two "Line" drawings per target: a black background bar
	          spanning the full height of the box on the left edge, and a colour-lerped
	          (red→green) fill line proportional to current HP. Toggled by the new
	          "Health Bars (drawn)" ESP toggle.
	ADDED:    Corner Box mode — 8 "Line" drawings per target forming L-shaped brackets
	          at each corner of the box, rainbow-cycled and auto-thickness-scaled by
	          distance, matching Blissful's Corner 2D Box ESP exactly.
	FIXED:    Double Jump impulse; ESP modifier rebuild (see original header notes).
	NOTE:     Drawing API requires an executor environment; panel loads fine without
	          it — ESP toggles will notify and no-op in a plain Studio LocalScript.
	================================================================================
]]

----------------------------------------------------------------------------
-- SERVICES
----------------------------------------------------------------------------
local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local Lighting         = game:GetService("Lighting")
local Workspace        = game:GetService("Workspace")
local HttpService      = game:GetService("HttpService")
local CollectionService= game:GetService("CollectionService")
local Stats            = game:GetService("Stats")
local SoundService     = game:GetService("SoundService")
local PhysicsService   = game:GetService("PhysicsService")
local StarterGui       = game:GetService("StarterGui")

local LocalPlayer = Players.LocalPlayer
local Camera      = Workspace.CurrentCamera
local Mouse       = LocalPlayer:GetMouse()

----------------------------------------------------------------------------
-- SETTINGS PERSISTENCE
----------------------------------------------------------------------------
local SETTINGS_FILE = "DevAdminPanel_Settings.json"

local function canPersist()
	return typeof(writefile) == "function"
		and typeof(readfile)  == "function"
		and typeof(isfile)    == "function"
end

local DEFAULT_SETTINGS = {
	Theme            = "Midnight",
	ToggleKey        = "RightControl",
	PaletteKey       = "P",
	WalkSpeed        = 16,
	JumpPower        = 50,
	FlySpeed         = 50,
	FOV              = 70,
	NotificationsOn  = true,
	OverlayOn        = true,
	GravityValue     = 196.2,
	SprintSpeed      = 28,
	DashDistance     = 40,
	TPDistance       = 12,
	SlowMoFactor     = 0.3,
	CamShakeAmt      = 0.5,
	CharScale        = 1,
	Keybinds         = {},
	WindowPos        = nil,
	Favorites        = {},
	Profiles         = {},
	SavedLocations   = {},
	NotifyHistory    = {},
}

local Settings
local function loadSettings()
	if canPersist() and isfile(SETTINGS_FILE) then
		local ok, data = pcall(function()
			return HttpService:JSONDecode(readfile(SETTINGS_FILE))
		end)
		if ok and type(data) == "table" then
			for k, v in pairs(DEFAULT_SETTINGS) do
				if data[k] == nil then data[k] = v end
			end
			return data
		end
	end
	local copy = {}
	for k, v in pairs(DEFAULT_SETTINGS) do copy[k] = v end
	return copy
end
Settings = loadSettings()

local _saveQueued = false
local function saveSettings()
	if not canPersist() then return end
	if _saveQueued then return end
	_saveQueued = true
	task.delay(0.5, function()
		pcall(function()
			writefile(SETTINGS_FILE, HttpService:JSONEncode(Settings))
		end)
		_saveQueued = false
	end)
end

----------------------------------------------------------------------------
-- UTIL
----------------------------------------------------------------------------
local Util = {}

function Util.New(class, props, children)
	local inst = Instance.new(class)
	if props then
		for k, v in pairs(props) do
			if k ~= "Parent" then inst[k] = v end
		end
	end
	if children then
		for _, c in ipairs(children) do c.Parent = inst end
	end
	if props and props.Parent then inst.Parent = props.Parent end
	return inst
end

function Util.Corner(r, p)
	return Util.New("UICorner", { CornerRadius = UDim.new(0, r or 8), Parent = p })
end

function Util.Stroke(p, color, thick, trans)
	return Util.New("UIStroke", {
		Color = color or Color3.fromRGB(255,255,255),
		Thickness = thick or 1,
		Transparency = trans or 0.7,
		ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
		Parent = p,
	})
end

function Util.Gradient(p, colorSeq, rot, transSeq)
	return Util.New("UIGradient", {
		Color = colorSeq, Rotation = rot or 90,
		Transparency = transSeq, Parent = p,
	})
end

function Util.Pad(p, l, r, t, b)
	return Util.New("UIPadding", {
		PaddingLeft   = UDim.new(0, l or 0),
		PaddingRight  = UDim.new(0, r or l or 0),
		PaddingTop    = UDim.new(0, t or l or 0),
		PaddingBottom = UDim.new(0, b or t or l or 0),
		Parent = p,
	})
end

function Util.Tween(inst, props, time, style, dir)
	local info = TweenInfo.new(time or 0.2,
		style or Enum.EasingStyle.Quint,
		dir or Enum.EasingDirection.Out)
	local tw = TweenService:Create(inst, info, props)
	tw:Play()
	return tw
end

function Util.MakeDraggable(handle, target, onEnd)
	local dragging, dragStart, startPos = false, nil, nil
	handle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch then
			dragging  = true
			dragStart = input.Position
			startPos  = target.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
					if onEnd then onEnd(target.Position) end
				end
			end)
		end
	end)
	handle.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
			or input.UserInputType == Enum.UserInputType.Touch) then
			local delta = input.Position - dragStart
			target.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + delta.X,
				startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)
end

function Util.DeepCopy(t)
	if type(t) ~= "table" then return t end
	local copy = {}
	for k, v in pairs(t) do copy[k] = Util.DeepCopy(v) end
	return copy
end

function Util.Round(n, dec)
	local m = 10^(dec or 0)
	return math.floor(n * m + 0.5) / m
end

----------------------------------------------------------------------------
-- THEME SYSTEM
----------------------------------------------------------------------------
local Theme = {}
Theme.Presets = {
	Midnight = {
		Background = Color3.fromRGB(18,19,26),
		Panel      = Color3.fromRGB(24,26,35),
		Elevated   = Color3.fromRGB(31,33,44),
		Accent     = Color3.fromRGB(108,99,255),
		AccentSoft = Color3.fromRGB(86,78,214),
		Text       = Color3.fromRGB(235,235,245),
		SubText    = Color3.fromRGB(150,152,168),
		Good       = Color3.fromRGB(86,214,130),
		Bad        = Color3.fromRGB(235,89,89),
	},
	Crimson = {
		Background = Color3.fromRGB(20,16,18),
		Panel      = Color3.fromRGB(28,21,24),
		Elevated   = Color3.fromRGB(38,28,32),
		Accent     = Color3.fromRGB(224,68,92),
		AccentSoft = Color3.fromRGB(180,52,74),
		Text       = Color3.fromRGB(240,235,236),
		SubText    = Color3.fromRGB(165,150,153),
		Good       = Color3.fromRGB(86,214,130),
		Bad        = Color3.fromRGB(235,89,89),
	},
	Arctic = {
		Background = Color3.fromRGB(16,20,24),
		Panel      = Color3.fromRGB(22,28,34),
		Elevated   = Color3.fromRGB(29,37,45),
		Accent     = Color3.fromRGB(64,188,224),
		AccentSoft = Color3.fromRGB(48,150,184),
		Text       = Color3.fromRGB(232,242,245),
		SubText    = Color3.fromRGB(145,163,170),
		Good       = Color3.fromRGB(86,214,130),
		Bad        = Color3.fromRGB(235,89,89),
	},
	Emerald = {
		Background = Color3.fromRGB(15,20,18),
		Panel      = Color3.fromRGB(20,28,24),
		Elevated   = Color3.fromRGB(27,37,32),
		Accent     = Color3.fromRGB(63,207,142),
		AccentSoft = Color3.fromRGB(46,168,114),
		Text       = Color3.fromRGB(230,240,234),
		SubText    = Color3.fromRGB(146,165,154),
		Good       = Color3.fromRGB(86,214,130),
		Bad        = Color3.fromRGB(235,89,89),
	},
	Sunset = {
		Background = Color3.fromRGB(20,15,18),
		Panel      = Color3.fromRGB(28,20,24),
		Elevated   = Color3.fromRGB(38,26,32),
		Accent     = Color3.fromRGB(255,120,60),
		AccentSoft = Color3.fromRGB(210,90,40),
		Text       = Color3.fromRGB(245,235,230),
		SubText    = Color3.fromRGB(170,150,145),
		Good       = Color3.fromRGB(86,214,130),
		Bad        = Color3.fromRGB(235,89,89),
	},
}
Theme.Current    = Theme.Presets[Settings.Theme] or Theme.Presets.Midnight
Theme._listeners = {}

function Theme.OnChange(fn)
	table.insert(Theme._listeners, fn)
	fn(Theme.Current)
end

function Theme.Set(name)
	local preset = Theme.Presets[name]
	if not preset then return end
	Theme.Current = preset
	Settings.Theme = name
	saveSettings()
	for _, fn in ipairs(Theme._listeners) do pcall(fn, preset) end
end

----------------------------------------------------------------------------
-- ROOT GUI
----------------------------------------------------------------------------
local ScreenGui = Util.New("ScreenGui", {
	Name            = "DevAdminPanel",
	ResetOnSpawn    = false,
	ZIndexBehavior  = Enum.ZIndexBehavior.Sibling,
	DisplayOrder    = 999,
	IgnoreGuiInset  = true,
})
if typeof(gethui) == "function" then
	ScreenGui.Parent = gethui()
else
	ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

local BlurEffect = Util.New("BlurEffect", { Size = 0, Parent = Lighting })
local _blurEnabled = false
local function setBlur(v)
	_blurEnabled = v
	Util.Tween(BlurEffect, { Size = v and 12 or 0 }, 0.3)
end

----------------------------------------------------------------------------
-- NOTIFICATION SYSTEM
----------------------------------------------------------------------------
local Notify = {}
do
	local container = Util.New("Frame", {
		Name                  = "Notifications",
		BackgroundTransparency= 1,
		AnchorPoint           = Vector2.new(1,1),
		Position              = UDim2.new(1,-20,1,-20),
		Size                  = UDim2.new(0,300,1,-40),
		Parent                = ScreenGui,
	})
	Util.New("UIListLayout", {
		Parent             = container,
		Padding            = UDim.new(0,8),
		HorizontalAlignment= Enum.HorizontalAlignment.Right,
		VerticalAlignment  = Enum.VerticalAlignment.Bottom,
		SortOrder          = Enum.SortOrder.LayoutOrder,
	})
	local kindColor = { Info="Accent", Success="Good", Error="Bad", Warn="Accent" }

	function Notify.Push(title, message, kind, duration)
		local entry = { title=title, message=message or "", kind=kind or "Info", time=os.time() }
		table.insert(Settings.NotifyHistory, 1, entry)
		if #Settings.NotifyHistory > 50 then table.remove(Settings.NotifyHistory) end
		saveSettings()
		if not Settings.NotificationsOn then return end
		kind     = kind or "Info"
		duration = duration or 3.5
		local card = Util.New("Frame", {
			BackgroundColor3 = Theme.Current.Panel,
			Size             = UDim2.new(1,0,0,0),
			AutomaticSize    = Enum.AutomaticSize.Y,
			ClipsDescendants = true,
			Parent           = container,
		})
		Util.Corner(10, card)
		Util.Stroke(card, Theme.Current.Accent, 1, 0.6)
		local bar = Util.New("Frame", {
			BackgroundColor3 = Theme.Current[kindColor[kind]] or Theme.Current.Accent,
			Size             = UDim2.new(0,4,1,0),
			Parent           = card,
		})
		Util.Corner(2, bar)
		Util.Pad(card, 14, 14, 10, 10)
		local titleLbl = Util.New("TextLabel", {
			BackgroundTransparency= 1,
			Size                  = UDim2.new(1,0,0,18),
			Font                  = Enum.Font.GothamBold,
			Text                  = title,
			TextColor3            = Theme.Current.Text,
			TextSize              = 14,
			TextXAlignment        = Enum.TextXAlignment.Left,
			Parent                = card,
		})
		local msgLbl = Util.New("TextLabel", {
			BackgroundTransparency= 1,
			Position              = UDim2.new(0,0,0,20),
			Size                  = UDim2.new(1,0,0,0),
			AutomaticSize         = Enum.AutomaticSize.Y,
			Font                  = Enum.Font.Gotham,
			Text                  = message or "",
			TextColor3            = Theme.Current.SubText,
			TextSize              = 13,
			TextWrapped           = true,
			TextXAlignment        = Enum.TextXAlignment.Left,
			Parent                = card,
		})
		card.BackgroundTransparency = 1
		titleLbl.TextTransparency   = 1
		msgLbl.TextTransparency     = 1
		bar.BackgroundTransparency  = 1
		card.Position = UDim2.new(0,40,0,0)
		Util.Tween(card,     { BackgroundTransparency=0, Position=UDim2.new(0,0,0,0) }, 0.25)
		Util.Tween(bar,      { BackgroundTransparency=0 }, 0.25)
		Util.Tween(titleLbl, { TextTransparency=0 }, 0.3)
		Util.Tween(msgLbl,   { TextTransparency=0.2 }, 0.3)
		task.delay(duration, function()
			if not card or not card.Parent then return end
			Util.Tween(card,     { BackgroundTransparency=1, Position=UDim2.new(0,40,0,0) }, 0.25)
			Util.Tween(titleLbl, { TextTransparency=1 }, 0.2)
			Util.Tween(msgLbl,   { TextTransparency=1 }, 0.2)
			task.wait(0.27)
			card:Destroy()
		end)
	end
end

----------------------------------------------------------------------------
-- STATE
----------------------------------------------------------------------------
local State = {
	PanelVisible    = true,
	Minimized       = false,
	SelectedPlayer  = nil,
	Flying          = false,
	Noclipping      = false,
	InfiniteJump    = false,
	DoubleJump      = false,
	_djUsed         = false,
	Spectating      = false,
	Freecamming     = false,
	Fullbright      = false,
	Invisible       = false,
	GodMode         = false,
	RainbowChar     = false,
	Sprinting       = false,
	SwimInAir       = false,
	WallClimbing    = false,
	Frozen          = false,
	FPLock          = false,
	ScreenshotMode  = false,
	XRay            = false,
	SlowMo          = false,
	SimulateLag     = false,
	SimulateFPS     = false,
	OrbitCam        = false,
	TrailActive     = false,
	AuraActive      = false,
	RemoteLogging   = false,
	ColVis          = false,
	HitVis          = false,
	PathVis         = false,
	_lastTeleportCF = nil,
	ESP = {
		-- target categories
		Players    = false,
		NPCs       = false,
		Items      = false,
		Objectives = false,
		Vehicles   = false,
		-- drawing styles (Blissful integration)
		Boxes      = false,   -- full 2-D quad box  (Blissful 2D Box ESP)
		CornerBox  = false,   -- L-bracket corner box (Blissful Corner 2D Box ESP)
		Tracers    = false,   -- line from screen-bottom to target feet
		HealthBars = false,   -- drawn HP bar on left edge of box
		-- text overlays
		Names      = false,
		Distances  = false,
		Health     = false,   -- text HP %
		-- other
		Skeleton   = false,
		TeamColor  = false,
	},
}

local _themedObjects = {}
local function themed(inst, prop, key)
	table.insert(_themedObjects, { inst=inst, prop=prop, key=key })
	inst[prop] = Theme.Current[key]
	return inst
end
Theme.OnChange(function(c)
	for _, e in ipairs(_themedObjects) do
		if e.inst and e.inst.Parent then
			Util.Tween(e.inst, { [e.prop] = c[e.key] }, 0.25)
		end
	end
end)

local function Character() return LocalPlayer.Character end
local function Humanoid()  local c=Character() return c and c:FindFirstChildOfClass("Humanoid") end
local function HRP()       local c=Character() return c and c:FindFirstChild("HumanoidRootPart") end

----------------------------------------------------------------------------
-- MAIN WINDOW SHELL
----------------------------------------------------------------------------
local Window = Util.New("Frame", {
	Name             = "Window",
	Size             = UDim2.new(0,860,0,540),
	Position         = UDim2.new(0.5,-430,0.5,-270),
	BackgroundColor3 = Theme.Current.Background,
	ClipsDescendants = false,
	Parent           = ScreenGui,
})
Util.Corner(14, Window)
themed(Window, "BackgroundColor3", "Background")
local windowStroke = Util.Stroke(Window, Theme.Current.Accent, 1, 0.75)
themed(windowStroke, "Color", "Accent")

if Settings.WindowPos then
	Window.Position = UDim2.new(0, Settings.WindowPos[1], 0, Settings.WindowPos[2])
end

Util.New("ImageLabel", {
	Name               = "Shadow",
	BackgroundTransparency = 1,
	Image              = "rbxassetid://1316045217",
	ImageColor3        = Color3.new(0,0,0),
	ImageTransparency  = 0.45,
	ScaleType          = Enum.ScaleType.Slice,
	SliceCenter        = Rect.new(10,10,118,118),
	Size               = UDim2.new(1,60,1,60),
	Position           = UDim2.new(0.5,0,0.5,6),
	AnchorPoint        = Vector2.new(0.5,0.5),
	ZIndex             = -1,
	Parent             = Window,
})

----------------------------------------------------------------------------
-- TITLE BAR
----------------------------------------------------------------------------
local TitleBar = Util.New("Frame", {
	Name             = "TitleBar",
	Size             = UDim2.new(1,0,0,46),
	BackgroundColor3 = Theme.Current.Panel,
	Parent           = Window,
})
Util.Corner(14, TitleBar)
themed(TitleBar, "BackgroundColor3", "Panel")
local _tbFill = Util.New("Frame", {
	BackgroundColor3 = Theme.Current.Panel,
	BorderSizePixel  = 0,
	Position         = UDim2.new(0,0,1,-14),
	Size             = UDim2.new(1,0,0,14),
	Parent           = TitleBar,
})
themed(_tbFill, "BackgroundColor3", "Panel")

local TitleDot = Util.New("Frame", {
	Size             = UDim2.new(0,10,0,10),
	Position         = UDim2.new(0,18,0.5,-5),
	BackgroundColor3 = Theme.Current.Accent,
	Parent           = TitleBar,
})
Util.Corner(5, TitleDot)
themed(TitleDot, "BackgroundColor3", "Accent")

local TitleText = Util.New("TextLabel", {
	BackgroundTransparency= 1,
	Position              = UDim2.new(0,38,0,0),
	Size                  = UDim2.new(1,-220,1,0),
	Font                  = Enum.Font.GothamBold,
	Text                  = "Developer Panel  //  Made By RONTHEBARON  |  ESP by Blissful4992",
	TextColor3            = Theme.Current.Text,
	TextSize              = 14,
	TextXAlignment        = Enum.TextXAlignment.Left,
	Parent                = TitleBar,
})
themed(TitleText, "TextColor3", "Text")

local function TitleButton(parent, xOffset, glyph, hoverColor)
	local btn = Util.New("TextButton", {
		Size             = UDim2.new(0,30,0,30),
		Position         = UDim2.new(1,xOffset,0.5,-15),
		BackgroundColor3 = Theme.Current.Elevated,
		Text             = glyph,
		Font             = Enum.Font.GothamBold,
		TextSize         = 13,
		TextColor3       = Theme.Current.SubText,
		AutoButtonColor  = false,
		Parent           = parent,
	})
	Util.Corner(8, btn)
	themed(btn, "BackgroundColor3", "Elevated")
	btn.MouseEnter:Connect(function() Util.Tween(btn, { BackgroundColor3 = hoverColor or Theme.Current.AccentSoft }, 0.15) end)
	btn.MouseLeave:Connect(function() Util.Tween(btn, { BackgroundColor3 = Theme.Current.Elevated }, 0.15) end)
	return btn
end

local BlurBtn     = TitleButton(TitleBar, -160, "◈")
local DockBtn     = TitleButton(TitleBar, -122, "⊞")
local MinimizeBtn = TitleButton(TitleBar, -82,  "—")
local CloseBtn    = TitleButton(TitleBar, -44,  "✕", Theme.Current.Bad)

Util.MakeDraggable(TitleBar, Window, function(pos)
	Settings.WindowPos = { pos.X.Offset, pos.Y.Offset }
	saveSettings()
end)

-- Tooltip
local Tooltip = Util.New("Frame", {
	BackgroundColor3 = Theme.Current.Elevated,
	Size             = UDim2.new(0,200,0,0),
	AutomaticSize    = Enum.AutomaticSize.Y,
	Visible          = false,
	ZIndex           = 200,
	Parent           = ScreenGui,
})
Util.Corner(6, Tooltip)
Util.Stroke(Tooltip, Theme.Current.Accent, 1, 0.5)
Util.Pad(Tooltip, 8,8,6,6)
themed(Tooltip, "BackgroundColor3", "Elevated")
local TooltipLabel = Util.New("TextLabel", {
	BackgroundTransparency= 1,
	Size                  = UDim2.new(1,0,0,0),
	AutomaticSize         = Enum.AutomaticSize.Y,
	Font                  = Enum.Font.Gotham,
	TextSize              = 12,
	TextColor3            = Theme.Current.SubText,
	TextWrapped           = true,
	Text                  = "",
	Parent                = Tooltip,
})
themed(TooltipLabel, "TextColor3", "SubText")

local function AttachTooltip(inst, text)
	inst.MouseEnter:Connect(function()
		TooltipLabel.Text = text
		Tooltip.Position  = UDim2.new(0, Mouse.X+14, 0, Mouse.Y+14)
		Tooltip.Visible   = true
	end)
	inst.MouseLeave:Connect(function() Tooltip.Visible = false end)
	inst.MouseMoved:Connect(function()
		Tooltip.Position = UDim2.new(0, Mouse.X+14, 0, Mouse.Y+14)
	end)
end

----------------------------------------------------------------------------
-- SIDEBAR + CONTENT
----------------------------------------------------------------------------
local Body = Util.New("Frame", {
	Name                  = "Body",
	Position              = UDim2.new(0,0,0,46),
	Size                  = UDim2.new(1,0,1,-46),
	BackgroundTransparency= 1,
	Parent                = Window,
})

local Sidebar = Util.New("Frame", {
	Name             = "Sidebar",
	Size             = UDim2.new(0,180,1,0),
	BackgroundColor3 = Theme.Current.Panel,
	Parent           = Body,
})
themed(Sidebar, "BackgroundColor3", "Panel")
Util.New("UIListLayout", { Parent=Sidebar, Padding=UDim.new(0,4), SortOrder=Enum.SortOrder.LayoutOrder })
Util.Pad(Sidebar, 10, 10, 12, 12)

local SidebarSearch = Util.New("TextBox", {
	Size                = UDim2.new(1,0,0,28),
	BackgroundColor3    = Theme.Current.Elevated,
	PlaceholderText     = "🔍 Search features...",
	Text                = "",
	Font                = Enum.Font.Gotham,
	TextSize            = 12,
	TextColor3          = Theme.Current.Text,
	PlaceholderColor3   = Theme.Current.SubText,
	ClearTextOnFocus    = false,
	LayoutOrder         = 0,
	Parent              = Sidebar,
})
Util.Corner(6, SidebarSearch)
Util.Pad(SidebarSearch, 8)
themed(SidebarSearch, "BackgroundColor3", "Elevated")

local ContentArea = Util.New("Frame", {
	Name                  = "ContentArea",
	Position              = UDim2.new(0,180,0,0),
	Size                  = UDim2.new(1,-180,1,0),
	BackgroundTransparency= 1,
	Parent                = Body,
})
Util.Pad(ContentArea, 16, 16, 14, 14)

local Pages     = {}
local ActivePage = nil
local _pageHistory = {}

local function SwitchPage(name)
	if not Pages[name] then return end
	if ActivePage == name then return end
	for pname, p in pairs(Pages) do
		local active = pname == name
		p.page.Visible = active
		Util.Tween(p.button, {
			BackgroundColor3 = active and Theme.Current.AccentSoft or Theme.Current.Panel,
		}, 0.15)
		p.label.TextColor3 = active and Theme.Current.Text or Theme.Current.SubText
	end
	ActivePage = name
	table.insert(_pageHistory, 1, name)
	if #_pageHistory > 20 then table.remove(_pageHistory) end
end

local function CreateTab(name, icon, order)
	local btn = Util.New("TextButton", {
		Size             = UDim2.new(1,0,0,34),
		BackgroundColor3 = Theme.Current.Panel,
		AutoButtonColor  = false,
		Text             = "",
		LayoutOrder      = order,
		Parent           = Sidebar,
	})
	Util.Corner(8, btn)
	local label = Util.New("TextLabel", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,-16,1,0),
		Position              = UDim2.new(0,16,0,0),
		Font                  = Enum.Font.GothamMedium,
		Text                  = (icon and (icon.." ") or "") .. name,
		TextColor3            = Theme.Current.SubText,
		TextSize              = 13,
		TextXAlignment        = Enum.TextXAlignment.Left,
		Parent                = btn,
	})
	local page = Util.New("ScrollingFrame", {
		Name                = name.."Page",
		BackgroundTransparency= 1,
		Size                = UDim2.new(1,0,1,0),
		CanvasSize          = UDim2.new(0,0,0,0),
		AutomaticCanvasSize = Enum.AutomaticSize.Y,
		ScrollBarThickness  = 4,
		ScrollBarImageColor3= Theme.Current.Accent,
		Visible             = false,
		Parent              = ContentArea,
	})
	Util.New("UIListLayout", {
		Parent    = page, Padding = UDim.new(0,10),
		SortOrder = Enum.SortOrder.LayoutOrder,
	})
	btn.MouseButton1Click:Connect(function() SwitchPage(name) end)
	Pages[name] = { button=btn, label=label, page=page }
	return page
end

----------------------------------------------------------------------------
-- FEATURE SEARCH OVERLAY
----------------------------------------------------------------------------
local SearchOverlay = Util.New("ScrollingFrame", {
	Name                = "SearchOverlay",
	BackgroundTransparency= 1,
	Size                = UDim2.new(1,0,1,0),
	CanvasSize          = UDim2.new(0,0,0,0),
	AutomaticCanvasSize = Enum.AutomaticSize.Y,
	ScrollBarThickness  = 4,
	ScrollBarImageColor3= Theme.Current.Accent,
	Visible             = false,
	Parent              = ContentArea,
})
Util.New("UIListLayout", { Parent=SearchOverlay, Padding=UDim.new(0,6), SortOrder=Enum.SortOrder.LayoutOrder })

local AllFeatureLabels = {}
local function RegFeature(name, tab, desc)
	table.insert(AllFeatureLabels, { name=name, tab=tab, description=desc or "" })
end

SidebarSearch:GetPropertyChangedSignal("Text"):Connect(function()
	local q = SidebarSearch.Text:lower()
	for _, c in ipairs(SearchOverlay:GetChildren()) do
		if c:IsA("GuiObject") then c:Destroy() end
	end
	if q == "" then
		SearchOverlay.Visible = false
		if ActivePage then
			for _, p in pairs(Pages) do
				p.page.Visible = (Pages[ActivePage] == p)
			end
		end
		return
	end
	SearchOverlay.Visible = true
	for _, p in pairs(Pages) do p.page.Visible = false end
	local count = 0
	for _, feat in ipairs(AllFeatureLabels) do
		if feat.name:lower():find(q, 1, true) or feat.description:lower():find(q, 1, true) then
			count += 1
			local row = Util.New("Frame", {
				BackgroundColor3 = Theme.Current.Panel,
				Size             = UDim2.new(1,-8,0,44),
				LayoutOrder      = count,
				Parent           = SearchOverlay,
			})
			Util.Corner(8, row)
			Util.Pad(row, 12,12,8,8)
			Util.New("TextLabel", {
				BackgroundTransparency= 1,
				Size                  = UDim2.new(1,-60,0,18),
				Font                  = Enum.Font.GothamBold,
				Text                  = feat.name,
				TextColor3            = Theme.Current.Text,
				TextSize              = 13,
				TextXAlignment        = Enum.TextXAlignment.Left,
				Parent                = row,
			})
			Util.New("TextLabel", {
				BackgroundTransparency= 1,
				Position              = UDim2.new(0,0,0,20),
				Size                  = UDim2.new(1,-60,0,14),
				Font                  = Enum.Font.Gotham,
				Text                  = feat.description ~= "" and feat.description or ("Tab: "..feat.tab),
				TextColor3            = Theme.Current.SubText,
				TextSize              = 11,
				TextXAlignment        = Enum.TextXAlignment.Left,
				Parent                = row,
			})
			local goBtn = Util.New("TextButton", {
				Size             = UDim2.new(0,52,0,26),
				Position         = UDim2.new(1,-52,0.5,-13),
				BackgroundColor3 = Theme.Current.Accent,
				Text             = "Go →",
				Font             = Enum.Font.GothamBold,
				TextSize         = 11,
				TextColor3       = Color3.new(1,1,1),
				AutoButtonColor  = false,
				Parent           = row,
			})
			Util.Corner(6, goBtn)
			local tabName = feat.tab
			goBtn.MouseButton1Click:Connect(function()
				SidebarSearch.Text    = ""
				SearchOverlay.Visible = false
				SwitchPage(tabName)
			end)
		end
	end
	if count == 0 then
		Util.New("TextLabel", {
			BackgroundTransparency= 1,
			Size                  = UDim2.new(1,0,0,40),
			Font                  = Enum.Font.Gotham,
			Text                  = "No features found.",
			TextColor3            = Theme.Current.SubText,
			TextSize              = 13,
			Parent                = SearchOverlay,
		})
	end
end)

----------------------------------------------------------------------------
-- REUSABLE COMPONENT FACTORY
----------------------------------------------------------------------------
local Components = {}
local Favorites  = Settings.Favorites or {}

function Components.Section(parent, title, order)
	local card = Util.New("Frame", {
		BackgroundColor3 = Theme.Current.Panel,
		Size             = UDim2.new(1,-8,0,0),
		AutomaticSize    = Enum.AutomaticSize.Y,
		LayoutOrder      = order or 0,
		Parent           = parent,
	})
	Util.Corner(10, card)
	themed(card, "BackgroundColor3", "Panel")
	Util.Pad(card, 14, 14, 12, 12)
	Util.New("UIListLayout", {
		Parent    = card, Padding = UDim.new(0,10),
		SortOrder = Enum.SortOrder.LayoutOrder,
	})
	if title then
		local hdr = Util.New("Frame", {
			BackgroundTransparency= 1,
			Size                  = UDim2.new(1,0,0,20),
			LayoutOrder           = -1,
			Parent                = card,
		})
		Util.New("TextLabel", {
			BackgroundTransparency= 1,
			Size                  = UDim2.new(1,0,1,0),
			Font                  = Enum.Font.GothamBold,
			Text                  = title,
			TextColor3            = Theme.Current.Text,
			TextSize              = 14,
			TextXAlignment        = Enum.TextXAlignment.Left,
			Parent                = hdr,
		})
	end
	return card
end

function Components.Toggle(parent, text, default, callback, order, tooltip)
	local row = Util.New("Frame", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,0,0,26),
		LayoutOrder           = order or 0,
		Parent                = parent,
	})
	local starBtn = Util.New("TextButton", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(0,20,1,0),
		Font                  = Enum.Font.GothamBold,
		Text                  = Favorites[text] and "★" or "☆",
		TextColor3            = Favorites[text] and Theme.Current.Accent or Theme.Current.SubText,
		TextSize              = 14,
		AutoButtonColor       = false,
		Parent                = row,
	})
	starBtn.MouseButton1Click:Connect(function()
		Favorites[text] = not Favorites[text]
		starBtn.Text      = Favorites[text] and "★" or "☆"
		starBtn.TextColor3= Favorites[text] and Theme.Current.Accent or Theme.Current.SubText
		Settings.Favorites= Favorites
		saveSettings()
	end)
	local lbl = Util.New("TextLabel", {
		BackgroundTransparency= 1,
		Position              = UDim2.new(0,22,0,0),
		Size                  = UDim2.new(1,-72,1,0),
		Font                  = Enum.Font.Gotham,
		Text                  = text,
		TextColor3            = Theme.Current.SubText,
		TextSize              = 13,
		TextXAlignment        = Enum.TextXAlignment.Left,
		Parent                = row,
	})
	if tooltip then AttachTooltip(lbl, tooltip) end
	local track = Util.New("TextButton", {
		Text             = "",
		AutoButtonColor  = false,
		Size             = UDim2.new(0,42,0,22),
		Position         = UDim2.new(1,-42,0.5,-11),
		BackgroundColor3 = default and Theme.Current.Accent or Theme.Current.Elevated,
		Parent           = row,
	})
	Util.Corner(11, track)
	local knob = Util.New("Frame", {
		Size             = UDim2.new(0,18,0,18),
		Position         = default and UDim2.new(1,-20,0.5,-9) or UDim2.new(0,2,0.5,-9),
		BackgroundColor3 = Color3.new(1,1,1),
		Parent           = track,
	})
	Util.Corner(9, knob)
	local state = default
	local function refresh(animated)
		local t = animated and Util.Tween or function(i,p) for k,v in pairs(p) do i[k]=v end end
		t(track, { BackgroundColor3 = state and Theme.Current.Accent or Theme.Current.Elevated }, 0.15)
		t(knob,  { Position = state and UDim2.new(1,-20,0.5,-9) or UDim2.new(0,2,0.5,-9) }, 0.15)
	end
	track.MouseButton1Click:Connect(function()
		state = not state
		refresh(true)
		callback(state)
	end)
	Theme.OnChange(function() refresh(false) end)
	return {
		Set = function(v) state=v; refresh(true) end,
		Get = function() return state end,
	}
end

function Components.Slider(parent, text, min, max, default, callback, order, tooltip)
	local row = Util.New("Frame", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,0,0,40),
		LayoutOrder           = order or 0,
		Parent                = parent,
	})
	local lbl = Util.New("TextLabel", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,-50,0,16),
		Font                  = Enum.Font.Gotham,
		Text                  = text,
		TextColor3            = Theme.Current.SubText,
		TextSize              = 13,
		TextXAlignment        = Enum.TextXAlignment.Left,
		Parent                = row,
	})
	if tooltip then AttachTooltip(lbl, tooltip) end
	local valueLbl = Util.New("TextLabel", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(0,50,0,16),
		Position              = UDim2.new(1,-50,0,0),
		Font                  = Enum.Font.GothamBold,
		Text                  = tostring(default),
		TextColor3            = Theme.Current.Text,
		TextSize              = 13,
		TextXAlignment        = Enum.TextXAlignment.Right,
		Parent                = row,
	})
	local bar = Util.New("Frame", {
		Position         = UDim2.new(0,0,0,24),
		Size             = UDim2.new(1,0,0,6),
		BackgroundColor3 = Theme.Current.Elevated,
		Parent           = row,
	})
	Util.Corner(3, bar)
	local fillRatio = math.clamp((default-min)/(max-min), 0, 1)
	local fill = Util.New("Frame", {
		Size             = UDim2.new(fillRatio,0,1,0),
		BackgroundColor3 = Theme.Current.Accent,
		Parent           = bar,
	})
	Util.Corner(3, fill)
	local knob = Util.New("Frame", {
		Size        = UDim2.new(0,14,0,14),
		AnchorPoint = Vector2.new(0.5,0.5),
		Position    = UDim2.new(fillRatio,0,0.5,0),
		BackgroundColor3= Color3.new(1,1,1),
		Parent      = bar,
	})
	Util.Corner(7, knob)
	local dragging = false
	local function setFromX(x)
		local rel = math.clamp((x-bar.AbsolutePosition.X)/bar.AbsoluteSize.X, 0, 1)
		local val = Util.Round(min + (max-min)*rel, 1)
		fill.Size       = UDim2.new(rel,0,1,0)
		knob.Position   = UDim2.new(rel,0,0.5,0)
		valueLbl.Text   = tostring(val)
		callback(val)
	end
	bar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true; setFromX(input.Position.X)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			setFromX(input.Position.X)
		end
	end)
	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
		end
	end)
	Theme.OnChange(function(c)
		bar.BackgroundColor3  = c.Elevated
		fill.BackgroundColor3 = c.Accent
	end)
	return {
		Set = function(v)
			local rel = math.clamp((v-min)/(max-min), 0, 1)
			fill.Size       = UDim2.new(rel,0,1,0)
			knob.Position   = UDim2.new(rel,0,0.5,0)
			valueLbl.Text   = tostring(v)
		end,
	}
end

function Components.Button(parent, text, callback, order, tooltip)
	local btn = Util.New("TextButton", {
		Size             = UDim2.new(1,0,0,32),
		BackgroundColor3 = Theme.Current.Accent,
		Text             = text,
		Font             = Enum.Font.GothamBold,
		TextSize         = 13,
		TextColor3       = Color3.new(1,1,1),
		AutoButtonColor  = false,
		LayoutOrder      = order or 0,
		Parent           = parent,
	})
	Util.Corner(8, btn)
	themed(btn, "BackgroundColor3", "Accent")
	btn.MouseEnter:Connect(function() Util.Tween(btn, { BackgroundColor3 = Theme.Current.AccentSoft }, 0.12) end)
	btn.MouseLeave:Connect(function() Util.Tween(btn, { BackgroundColor3 = Theme.Current.Accent }, 0.12) end)
	btn.MouseButton1Click:Connect(callback)
	if tooltip then AttachTooltip(btn, tooltip) end
	return btn
end

function Components.DangerButton(parent, text, callback, order)
	local btn = Util.New("TextButton", {
		Size             = UDim2.new(1,0,0,32),
		BackgroundColor3 = Theme.Current.Bad,
		Text             = text,
		Font             = Enum.Font.GothamBold,
		TextSize         = 13,
		TextColor3       = Color3.new(1,1,1),
		AutoButtonColor  = false,
		LayoutOrder      = order or 0,
		Parent           = parent,
	})
	Util.Corner(8, btn)
	btn.MouseButton1Click:Connect(callback)
	return btn
end

function Components.Label(parent, text, order, small)
	return Util.New("TextLabel", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,0,0, small and 14 or 18),
		Font                  = Enum.Font.Gotham,
		Text                  = text,
		TextColor3            = Theme.Current.SubText,
		TextSize              = small and 11 or 13,
		TextXAlignment        = Enum.TextXAlignment.Left,
		LayoutOrder           = order or 0,
		Parent                = parent,
	})
end

function Components.LogBox(parent, order)
	local frame = Util.New("Frame", {
		BackgroundColor3 = Theme.Current.Elevated,
		Size             = UDim2.new(1,0,0,120),
		LayoutOrder      = order or 0,
		Parent           = parent,
	})
	Util.Corner(8, frame)
	themed(frame, "BackgroundColor3", "Elevated")
	local scroll = Util.New("ScrollingFrame", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,0,1,0),
		CanvasSize            = UDim2.new(0,0,0,0),
		AutomaticCanvasSize   = Enum.AutomaticSize.Y,
		ScrollBarThickness    = 3,
		Parent                = frame,
	})
	Util.Pad(scroll, 8,8,6,6)
	Util.New("UIListLayout", {
		Parent    = scroll, Padding = UDim.new(0,2), SortOrder = Enum.SortOrder.LayoutOrder,
	})
	local lineCount = 0
	local box = {
		frame  = frame,
		scroll = scroll,
		Append = function(msg, color)
			lineCount += 1
			Util.New("TextLabel", {
				BackgroundTransparency= 1,
				Size                  = UDim2.new(1,-4,0,14),
				Font                  = Enum.Font.Code,
				Text                  = msg,
				TextColor3            = color or Theme.Current.Text,
				TextSize              = 11,
				TextXAlignment        = Enum.TextXAlignment.Left,
				TextTruncate          = Enum.TextTruncate.AtEnd,
				LayoutOrder           = lineCount,
				Parent                = scroll,
			})
			task.defer(function()
				scroll.CanvasPosition = Vector2.new(0, scroll.AbsoluteCanvasSize.Y)
			end)
		end,
		Clear = function()
			for _, c in ipairs(scroll:GetChildren()) do
				if c:IsA("TextLabel") then c:Destroy() end
			end
			lineCount = 0
		end,
	}
	return box
end

function Components.KeybindButton(parent, featureId, defaultKeyName, order)
	local current = Settings.Keybinds[featureId] or defaultKeyName
	local btn = Util.New("TextButton", {
		Size             = UDim2.new(0,90,0,26),
		BackgroundColor3 = Theme.Current.Elevated,
		Text             = current,
		Font             = Enum.Font.GothamBold,
		TextSize         = 12,
		TextColor3       = Theme.Current.Text,
		AutoButtonColor  = false,
		LayoutOrder      = order or 0,
		Parent           = parent,
	})
	Util.Corner(6, btn)
	themed(btn, "BackgroundColor3", "Elevated")
	btn.MouseButton1Click:Connect(function()
		btn.Text = "..."
		local conn
		conn = UserInputService.InputBegan:Connect(function(input, gpe)
			if input.UserInputType == Enum.UserInputType.Keyboard then
				conn:Disconnect()
				local kn = input.KeyCode.Name
				Settings.Keybinds[featureId] = kn
				btn.Text = kn
				saveSettings()
				Notify.Push("Keybind Updated", featureId.." → "..kn, "Info", 2)
			end
		end)
	end)
	return btn
end

function Components.ColorPicker(parent, label, default, callback, order)
	local card = Components.Section(parent, label, order)
	local r,g,b = default.R*255, default.G*255, default.B*255
	local preview = Util.New("Frame", {
		Size             = UDim2.new(0,30,0,16),
		BackgroundColor3 = default,
		LayoutOrder      = 1,
		Parent           = card,
	})
	Util.Corner(4, preview)
	local function fire()
		local c = Color3.fromRGB(r,g,b)
		preview.BackgroundColor3 = c
		callback(c)
	end
	Components.Slider(card, "R", 0, 255, r, function(v) r=v; fire() end, 2)
	Components.Slider(card, "G", 0, 255, g, function(v) g=v; fire() end, 3)
	Components.Slider(card, "B", 0, 255, b, function(v) b=v; fire() end, 4)
	return card
end

----------------------------------------------------------------------------
-- FEATURES REGISTRY
----------------------------------------------------------------------------
local Features = {}

local function applyMovementStats()
	local hum = Humanoid()
	if not hum then return end
	hum.WalkSpeed = Settings.WalkSpeed
	hum.JumpPower = Settings.JumpPower
	Workspace.Gravity = Settings.GravityValue
end

LocalPlayer.CharacterAdded:Connect(function(char)
	task.wait(0.5)
	applyMovementStats()
	if Settings.CharScale and Settings.CharScale ~= 1 then
		local hum = char:FindFirstChildOfClass("Humanoid")
		if hum then
			local desc = hum:GetAppliedDescription()
			desc.HeadScale    = Settings.CharScale
			desc.BodyTypeScale= Settings.CharScale - 1
			hum:ApplyDescription(desc)
		end
	end
end)

----------------------------------------------------------------------------
-- FEATURE: Fly
----------------------------------------------------------------------------
local FlyConn
local flyKeys = { W=false, A=false, S=false, D=false, Up=false, Down=false }
local Fly = {
	Enable = function()
		local hum, root = Humanoid(), HRP()
		if not hum or not root then return end
		State.Flying       = true
		hum.PlatformStand  = true
		if FlyConn then FlyConn:Disconnect() end
		FlyConn = RunService.Heartbeat:Connect(function()
			if not State.Flying then return end
			local cam = Workspace.CurrentCamera
			local dir = Vector3.new()
			if flyKeys.W    then dir += cam.CFrame.LookVector end
			if flyKeys.S    then dir -= cam.CFrame.LookVector end
			if flyKeys.A    then dir -= cam.CFrame.RightVector end
			if flyKeys.D    then dir += cam.CFrame.RightVector end
			if flyKeys.Up   then dir += Vector3.new(0,1,0) end
			if flyKeys.Down then dir -= Vector3.new(0,1,0) end
			if dir.Magnitude > 0 then dir = dir.Unit * Settings.FlySpeed end
			local r = HRP()
			if r then r.AssemblyLinearVelocity = dir end
		end)
	end,
	Disable = function()
		State.Flying = false
		if FlyConn then FlyConn:Disconnect(); FlyConn=nil end
		local hum = Humanoid()
		if hum then hum.PlatformStand = false end
	end,
}
Features.Fly = Fly
RegFeature("Fly", "Movement", "Enable flight — WASD + Space/Shift")

----------------------------------------------------------------------------
-- FEATURE: Noclip
----------------------------------------------------------------------------
local NoclipConn
local Noclip = {
	Enable = function()
		State.Noclipping = true
		if NoclipConn then NoclipConn:Disconnect() end
		NoclipConn = RunService.Stepped:Connect(function()
			local char = Character()
			if not char then return end
			for _, part in ipairs(char:GetDescendants()) do
				if part:IsA("BasePart") then part.CanCollide = false end
			end
		end)
	end,
	Disable = function()
		State.Noclipping = false
		if NoclipConn then NoclipConn:Disconnect(); NoclipConn=nil end
		local char = Character()
		if char then
			for _, part in ipairs(char:GetDescendants()) do
				if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
					part.CanCollide = true
				end
			end
		end
	end,
}
Features.Noclip = Noclip
RegFeature("Noclip", "Movement", "Walk through walls and parts")

----------------------------------------------------------------------------
-- FEATURE: Infinite Jump / Double Jump
----------------------------------------------------------------------------
local InfJump   = { Enable=function() State.InfiniteJump=true end, Disable=function() State.InfiniteJump=false end }
local DoubleJump= { Enable=function() State.DoubleJump=true; State._djUsed=false end, Disable=function() State.DoubleJump=false end }
Features.InfiniteJump = InfJump
Features.DoubleJump   = DoubleJump
RegFeature("Infinite Jump", "Movement", "Jump indefinitely while in the air")
RegFeature("Double Jump",   "Movement", "Jump a second time while in the air")

RunService.Heartbeat:Connect(function()
	if State.DoubleJump then
		local hum = Humanoid()
		if hum and hum:GetState() == Enum.HumanoidStateType.Landed then
			State._djUsed = false
		end
	end
end)

UserInputService.JumpRequest:Connect(function()
	local hum = Humanoid()
	if not hum then return end
	if State.InfiniteJump then
		local root = HRP()
		if root then
			local vel = root.AssemblyLinearVelocity
			root.AssemblyLinearVelocity = Vector3.new(vel.X, hum.JumpPower > 0 and hum.JumpPower or 50, vel.Z)
		end
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
		return
	end
	if State.DoubleJump and not State._djUsed then
		local st = hum:GetState()
		if st == Enum.HumanoidStateType.Freefall or st == Enum.HumanoidStateType.Jumping then
			State._djUsed = true
			local root = HRP()
			if root then
				local vel = root.AssemblyLinearVelocity
				root.AssemblyLinearVelocity = Vector3.new(vel.X, hum.JumpPower > 0 and hum.JumpPower or 50, vel.Z)
			end
			hum:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

----------------------------------------------------------------------------
-- FEATURE: Sprint
----------------------------------------------------------------------------
local Sprint = {
	Toggle = function(on)
		State.Sprinting = on
		local hum = Humanoid()
		if hum then hum.WalkSpeed = on and Settings.SprintSpeed or Settings.WalkSpeed end
	end,
}
Features.Sprint = Sprint
RegFeature("Sprint Toggle", "Movement", "Hold Shift to sprint (configurable speed)")

local _shiftHeld = false
UserInputService.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	local k = input.KeyCode
	if k == Enum.KeyCode.LeftShift and not State.Flying then _shiftHeld=true; Sprint.Toggle(true) end
	if k == Enum.KeyCode.W     then flyKeys.W    = true end
	if k == Enum.KeyCode.A     then flyKeys.A    = true end
	if k == Enum.KeyCode.S     then flyKeys.S    = true end
	if k == Enum.KeyCode.D     then flyKeys.D    = true end
	if k == Enum.KeyCode.Space then flyKeys.Up   = true end
	if k == Enum.KeyCode.LeftShift and State.Flying then flyKeys.Down = true end
end)
UserInputService.InputEnded:Connect(function(input)
	local k = input.KeyCode
	if k == Enum.KeyCode.LeftShift then
		_shiftHeld = false
		if not State.Flying then Sprint.Toggle(false) end
	end
	if k == Enum.KeyCode.W     then flyKeys.W    = false end
	if k == Enum.KeyCode.A     then flyKeys.A    = false end
	if k == Enum.KeyCode.S     then flyKeys.S    = false end
	if k == Enum.KeyCode.D     then flyKeys.D    = false end
	if k == Enum.KeyCode.Space then flyKeys.Up   = false end
	if k == Enum.KeyCode.LeftShift then flyKeys.Down = false end
end)

----------------------------------------------------------------------------
-- FEATURE: Dash
----------------------------------------------------------------------------
local _dashCooldown = false
local Dash = {
	Do = function()
		if _dashCooldown then return end
		local root = HRP()
		if not root then return end
		_dashCooldown = true
		root.AssemblyLinearVelocity = root.CFrame.LookVector * Settings.DashDistance * 4
		task.delay(0.6, function() _dashCooldown = false end)
		Notify.Push("Dash", "Dashed forward!", "Info", 1)
	end,
}
Features.Dash = Dash
RegFeature("Dash Ability", "Movement", "Burst forward in look direction")

----------------------------------------------------------------------------
-- FEATURE: Wall Climb
----------------------------------------------------------------------------
local WallClimbConn
local WallClimb = {
	Enable = function()
		State.WallClimbing = true
		if WallClimbConn then WallClimbConn:Disconnect() end
		WallClimbConn = RunService.Heartbeat:Connect(function()
			if not State.WallClimbing then return end
			local hum, root = Humanoid(), HRP()
			if not hum or not root then return end
			local params = RaycastParams.new()
			params.FilterDescendantsInstances = { Character() }
			params.FilterType = Enum.RaycastFilterType.Exclude
			local result = Workspace:Raycast(root.Position, root.CFrame.LookVector * 2.5, params)
			if result and flyKeys.W then
				root.AssemblyLinearVelocity = Vector3.new(
					root.AssemblyLinearVelocity.X,
					Settings.WalkSpeed * 0.8,
					root.AssemblyLinearVelocity.Z)
			end
		end)
	end,
	Disable = function()
		State.WallClimbing = false
		if WallClimbConn then WallClimbConn:Disconnect(); WallClimbConn=nil end
	end,
}
Features.WallClimb = WallClimb
RegFeature("Wall Climb", "Movement", "Climb vertical surfaces while facing them")

----------------------------------------------------------------------------
-- FEATURE: Swim in Air
----------------------------------------------------------------------------
local SwimConn
local SwimInAir = {
	Enable = function()
		State.SwimInAir = true
		local hum = Humanoid()
		if hum then hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, false) end
		if SwimConn then SwimConn:Disconnect() end
		SwimConn = RunService.Stepped:Connect(function()
			local h = Humanoid()
			if h and State.SwimInAir then
				h:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)
				h:ChangeState(Enum.HumanoidStateType.Swimming)
			end
		end)
	end,
	Disable = function()
		State.SwimInAir = false
		if SwimConn then SwimConn:Disconnect(); SwimConn=nil end
		local hum = Humanoid()
		if hum then hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, true) end
	end,
}
Features.SwimInAir = SwimInAir
RegFeature("Swim in Air", "Movement", "Use swimming physics in open air")

----------------------------------------------------------------------------
-- FEATURE: Freeze / Gravity / Scale / Respawn
----------------------------------------------------------------------------
local Freeze = {
	Enable  = function() State.Frozen=true;  local r=HRP(); if r then r.Anchored=true end end,
	Disable = function() State.Frozen=false; local r=HRP(); if r then r.Anchored=false end end,
}
Features.Freeze = Freeze
RegFeature("Freeze Character", "Movement", "Anchor your character in place")

LocalPlayer.CharacterAdded:Connect(function(char)
	task.wait(0.1)
	if State.Frozen then
		local root = char:WaitForChild("HumanoidRootPart", 5)
		if root then root.Anchored = true end
	end
end)

local function applyGravity(v)
	Settings.GravityValue = v; Workspace.Gravity = v; saveSettings()
end
RegFeature("Gravity Changer", "Movement", "Change workspace gravity value")

local function applyCharScale(v)
	Settings.CharScale = v
	local hum = Humanoid()
	if hum then
		local desc = hum:GetAppliedDescription()
		desc.HeadScale       = v
		desc.BodyTypeScale   = math.clamp(v-1, 0, 1)
		desc.HeightScale     = v
		desc.WidthScale      = v
		desc.ProportionScale = math.clamp(v-1, 0, 1)
		hum:ApplyDescription(desc)
	end
	saveSettings()
end
RegFeature("Character Scaling", "Movement", "Scale your character size")

local function doRespawn() LocalPlayer:LoadCharacter() end
RegFeature("Respawn", "Movement", "Force-respawn your character")

----------------------------------------------------------------------------
-- FEATURE: Saved Teleport Locations
----------------------------------------------------------------------------
local _waypointParts = {}

local function destroyWaypointVisual(name)
	if _waypointParts[name] then
		pcall(function() _waypointParts[name].part:Destroy() end)
		_waypointParts[name] = nil
	end
end

local function createWaypointVisual(name, pos)
	destroyWaypointVisual(name)
	local beam = Util.New("Part", {
		Name         = "Waypoint_"..name,
		Anchored     = true,
		CanCollide   = false,
		CanQuery     = false,
		CanTouch     = false,
		Material     = Enum.Material.Neon,
		Color        = Theme.Current.Accent,
		Size         = Vector3.new(0.5, 14, 0.5),
		CFrame       = CFrame.new(pos + Vector3.new(0, 7, 0)),
		Transparency = 0.25,
		Parent       = Workspace,
	})
	Util.New("Part", {
		Name         = "WaypointRing_"..name,
		Anchored     = true,
		CanCollide   = false,
		CanQuery     = false,
		CanTouch     = false,
		Shape        = Enum.PartType.Cylinder,
		Material     = Enum.Material.Neon,
		Color        = Theme.Current.Accent,
		Size         = Vector3.new(0.3, 4, 4),
		CFrame       = CFrame.new(pos + Vector3.new(0, 0.2, 0)) * CFrame.Angles(0, 0, math.rad(90)),
		Transparency = 0.3,
		Parent       = beam,
	})
	local billboard = Util.New("BillboardGui", {
		Adornee      = beam,
		Size         = UDim2.new(0,170,0,40),
		StudsOffset  = Vector3.new(0, 9, 0),
		AlwaysOnTop  = true,
		Parent       = beam,
	})
	Util.New("TextLabel", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,0,1,0),
		Font                  = Enum.Font.GothamBold,
		Text                  = "📍 "..name,
		TextColor3            = Theme.Current.Accent,
		TextStrokeTransparency= 0,
		TextSize              = 16,
		Parent                = billboard,
	})
	task.spawn(function()
		local t = 0
		while beam and beam.Parent do
			t = t + task.wait(0.03)
			beam.CFrame = CFrame.new(pos + Vector3.new(0, 7 + math.sin(t*2)*0.4, 0))
		end
	end)
	_waypointParts[name] = { part=beam, billboard=billboard }
end

local function toggleWaypointVisual(name, pos, on)
	if on then createWaypointVisual(name, pos) else destroyWaypointVisual(name) end
end

local function saveTeleportLocation(name)
	local root = HRP()
	if not root then Notify.Push("Save Location", "No character!", "Error"); return end
	local pos = root.Position
	Settings.SavedLocations[name] = { pos.X, pos.Y, pos.Z, true }
	saveSettings()
	createWaypointVisual(name, pos)
	Notify.Push("Location Saved", name.." → "..math.floor(pos.X)..","..math.floor(pos.Y)..","..math.floor(pos.Z), "Success", 2)
end

local function teleportToSaved(name)
	local loc = Settings.SavedLocations[name]
	if not loc then return end
	local root = HRP()
	if not root then return end
	State._lastTeleportCF = root.CFrame
	root.CFrame = CFrame.new(loc[1], loc[2]+3, loc[3])
	Notify.Push("Teleported", "Moved to "..name, "Success", 2)
end

local function deleteSavedLocation(name)
	Settings.SavedLocations[name] = nil
	saveSettings()
	destroyWaypointVisual(name)
end

local function undoTeleport()
	if not State._lastTeleportCF then
		Notify.Push("Undo Teleport", "No previous location saved", "Warn"); return
	end
	local root = HRP()
	if not root then return end
	root.CFrame           = State._lastTeleportCF
	State._lastTeleportCF = nil
	Notify.Push("Undo Teleport", "Returned to previous location", "Success", 2)
end
RegFeature("Save Teleport Locations", "Movement", "Save & restore named positions with waypoint markers")
RegFeature("Undo Last Teleport",      "Movement", "Return to where you last teleported from")

for locName, loc in pairs(Settings.SavedLocations) do
	if loc[4] ~= false then
		createWaypointVisual(locName, Vector3.new(loc[1], loc[2], loc[3]))
	end
end

----------------------------------------------------------------------------
-- FEATURE: Freecam
----------------------------------------------------------------------------
local freecamConn
local freecamKeys = { W=false,A=false,S=false,D=false,Up=false,Down=false }
local savedCamType
local Freecam = {
	Enable = function()
		State.Freecamming = true
		savedCamType      = Camera.CameraType
		Camera.CameraType = Enum.CameraType.Scriptable
		if freecamConn then freecamConn:Disconnect() end
		freecamConn = RunService.RenderStepped:Connect(function(dt)
			if not State.Freecamming then return end
			local dir = Vector3.new()
			if freecamKeys.W    then dir += Camera.CFrame.LookVector end
			if freecamKeys.S    then dir -= Camera.CFrame.LookVector end
			if freecamKeys.A    then dir -= Camera.CFrame.RightVector end
			if freecamKeys.D    then dir += Camera.CFrame.RightVector end
			if freecamKeys.Up   then dir += Vector3.new(0,1,0) end
			if freecamKeys.Down then dir -= Vector3.new(0,1,0) end
			if dir.Magnitude > 0 then
				Camera.CFrame = Camera.CFrame + dir.Unit * Settings.FlySpeed * dt
			end
		end)
	end,
	Disable = function()
		State.Freecamming = false
		if freecamConn then freecamConn:Disconnect(); freecamConn=nil end
		Camera.CameraType = savedCamType or Enum.CameraType.Custom
	end,
}
Features.Freecam = Freecam
RegFeature("Freecam", "Movement", "Detach camera from character, fly it with WASD")

----------------------------------------------------------------------------
-- FEATURE: Cinematic Camera
----------------------------------------------------------------------------
local CinematicConn
local CinematicCam = {
	Active = false,
	Start = function()
		CinematicCam.Active = true
		Camera.CameraType   = Enum.CameraType.Scriptable
		local t = 0
		if CinematicConn then CinematicConn:Disconnect() end
		CinematicConn = RunService.RenderStepped:Connect(function(dt)
			t += dt * 0.3
			local center = HRP() and HRP().Position or Vector3.zero
			Camera.CFrame = CFrame.new(
				Vector3.new(center.X + math.cos(t)*20, center.Y+10, center.Z + math.sin(t)*20),
				center)
		end)
		Notify.Push("Cinematic Cam", "Press again to stop", "Info", 2)
	end,
	Stop = function()
		CinematicCam.Active = false
		if CinematicConn then CinematicConn:Disconnect(); CinematicConn=nil end
		Camera.CameraType = Enum.CameraType.Custom
	end,
}
RegFeature("Cinematic Camera", "Movement", "Orbiting cinematic camera around your character")

----------------------------------------------------------------------------
-- FEATURE: Camera Shake
----------------------------------------------------------------------------
local _shakeConn
local function startCameraShake(intensity, duration)
	if _shakeConn then _shakeConn:Disconnect() end
	local elapsed = 0
	_shakeConn = RunService.RenderStepped:Connect(function(dt)
		elapsed += dt
		if elapsed >= duration then _shakeConn:Disconnect(); _shakeConn=nil; return end
		local amt = intensity * (1 - elapsed/duration)
		Camera.CFrame = Camera.CFrame * CFrame.Angles(
			math.rad((math.random()-0.5)*amt*4),
			math.rad((math.random()-0.5)*amt*4),
			0)
	end)
end
RegFeature("Camera Shake", "Movement", "Trigger configurable camera shake")

----------------------------------------------------------------------------
-- FEATURE: Slow Motion
----------------------------------------------------------------------------
local SlowMo = {
	Enable  = function() State.SlowMo=true; Notify.Push("Slow Motion","Active (visual effect only)","Info",2) end,
	Disable = function() State.SlowMo=false end,
}
RegFeature("Slow-Motion Effect", "Movement", "Simulates slow-motion visually")

----------------------------------------------------------------------------
-- FEATURE: FOV / FP Lock / Orbit Cam / Screenshot Mode
----------------------------------------------------------------------------
local function applyFOV(v)
	Settings.FOV = v; Camera.FieldOfView = v; saveSettings()
end
RegFeature("Field of View", "Movement", "Change camera field of view")

local FPLock = {
	Enable  = function() State.FPLock=true;  LocalPlayer.CameraMinZoomDistance=0; LocalPlayer.CameraMaxZoomDistance=0 end,
	Disable = function() State.FPLock=false; LocalPlayer.CameraMinZoomDistance=0.5; LocalPlayer.CameraMaxZoomDistance=400 end,
}
RegFeature("First-Person Lock", "Movement", "Force first-person camera view")

local orbitConn
local _orbitAngle = 0
local OrbitCam = {
	Enable = function()
		State.OrbitCam    = true
		Camera.CameraType = Enum.CameraType.Scriptable
		if orbitConn then orbitConn:Disconnect() end
		orbitConn = RunService.RenderStepped:Connect(function(dt)
			if not State.OrbitCam then return end
			_orbitAngle += dt * 0.6
			local center = HRP() and HRP().Position or Vector3.zero
			local dist   = Settings.TPDistance
			Camera.CFrame= CFrame.new(
				Vector3.new(center.X + math.cos(_orbitAngle)*dist, center.Y+4, center.Z + math.sin(_orbitAngle)*dist),
				center)
		end)
	end,
	Disable = function()
		State.OrbitCam = false
		if orbitConn then orbitConn:Disconnect(); orbitConn=nil end
		Camera.CameraType = Enum.CameraType.Custom
	end,
}
RegFeature("Orbit Camera", "Movement", "Camera slowly orbits your character")

local ScreenshotMode = {
	Enable  = function() State.ScreenshotMode=true;  Window.Visible=false; Notify.Push("Screenshot Mode","UI hidden","Info",2) end,
	Disable = function() State.ScreenshotMode=false; Window.Visible=true end,
}
RegFeature("Screenshot Mode", "Movement", "Hide all panel UI for clean screenshots")

local dofEffect = Instance.new("DepthOfFieldEffect"); dofEffect.Parent=Lighting; dofEffect.Enabled=false
local function applyDOF(enabled, nearI, farI, focus)
	dofEffect.Enabled       = enabled
	dofEffect.NearIntensity = nearI  or 0
	dofEffect.FarIntensity  = farI   or 0.5
	dofEffect.FocusDistance = focus  or 50
	dofEffect.InFocusRadius = 15
end
RegFeature("Depth of Field", "Movement", "Apply depth-of-field blur to camera")

----------------------------------------------------------------------------
-- FEATURE: Fullbright
----------------------------------------------------------------------------
local savedLighting
local Fullbright = {
	Enable = function()
		if State.Fullbright then return end
		State.Fullbright = true
		savedLighting = {
			Brightness=Lighting.Brightness, ClockTime=Lighting.ClockTime,
			GlobalShadows=Lighting.GlobalShadows, FogEnd=Lighting.FogEnd,
			OutdoorAmbient=Lighting.OutdoorAmbient, Ambient=Lighting.Ambient,
		}
		Lighting.Brightness    = 2
		Lighting.ClockTime     = 14
		Lighting.GlobalShadows = false
		Lighting.FogEnd        = 1e8
		Lighting.OutdoorAmbient= Color3.fromRGB(180,180,180)
		Lighting.Ambient       = Color3.fromRGB(180,180,180)
	end,
	Disable = function()
		State.Fullbright = false
		if savedLighting then for k,v in pairs(savedLighting) do Lighting[k]=v end end
	end,
}
Features.Fullbright = Fullbright
RegFeature("Fullbright", "Visuals", "Maximize game lighting — see everything clearly")

----------------------------------------------------------------------------
-- FEATURE: Invisible / God Mode / Rainbow / Trail / Aura / Floating Text
----------------------------------------------------------------------------
local Invisible = {
	Enable = function()
		State.Invisible = true
		local char = Character(); if not char then return end
		for _, p in ipairs(char:GetDescendants()) do
			if p:IsA("BasePart") or p:IsA("Decal") then p.LocalTransparencyModifier=1 end
		end
	end,
	Disable = function()
		State.Invisible = false
		local char = Character(); if not char then return end
		for _, p in ipairs(char:GetDescendants()) do
			if p:IsA("BasePart") or p:IsA("Decal") then p.LocalTransparencyModifier=0 end
		end
	end,
}
Features.Invisible = Invisible
RegFeature("Invisible Character", "Visuals", "Make your character invisible locally")

LocalPlayer.CharacterAdded:Connect(function(char)
	if State.Invisible then
		char.DescendantAdded:Connect(function(part)
			if part:IsA("BasePart") or part:IsA("Decal") then part.LocalTransparencyModifier=1 end
		end)
	end
end)

local godConn
local GodModeVisual = {
	Enable = function()
		State.GodMode = true
		if godConn then godConn:Disconnect() end
		godConn = RunService.Heartbeat:Connect(function()
			local hum = Humanoid()
			if hum and State.GodMode then hum.MaxHealth=math.huge; hum.Health=math.huge end
		end)
		Notify.Push("God Mode", "Health locked to max (visual/local only)", "Success", 3)
	end,
	Disable = function()
		State.GodMode = false
		if godConn then godConn:Disconnect(); godConn=nil end
		local hum = Humanoid()
		if hum then hum.MaxHealth=100; hum.Health=100 end
	end,
}
Features.GodModeVisual = GodModeVisual
RegFeature("God Mode (Visual)", "Visuals", "Locks local health display to max")

local rainbowConn
local RainbowChar = {
	Enable = function()
		State.RainbowChar = true
		if rainbowConn then rainbowConn:Disconnect() end
		local h = 0
		rainbowConn = RunService.Heartbeat:Connect(function(dt)
			if not State.RainbowChar then return end
			h = (h + dt*60) % 360
			local col = Color3.fromHSV(h/360, 1, 1)
			local char = Character(); if not char then return end
			for _, p in ipairs(char:GetDescendants()) do
				if p:IsA("BasePart") then p.Color=col end
			end
		end)
	end,
	Disable = function()
		State.RainbowChar = false
		if rainbowConn then rainbowConn:Disconnect(); rainbowConn=nil end
	end,
}
Features.RainbowChar = RainbowChar
RegFeature("Rainbow Character", "Visuals", "Cycle your character through all colours")

local _activeTrail
local TrailCreator = {
	Enable = function(color1, color2)
		State.TrailActive = true
		local char = Character(); if not char then return end
		local root = char:FindFirstChild("HumanoidRootPart")
		local head = char:FindFirstChild("Head")
		if not root or not head then return end
		if _activeTrail then _activeTrail:Destroy() end
		local att0 = Instance.new("Attachment", root)
		local att1 = Instance.new("Attachment", head)
		_activeTrail = Util.New("Trail", {
			Attachment0  = att0,
			Attachment1  = att1,
			Color        = ColorSequence.new{
				ColorSequenceKeypoint.new(0, color1 or Theme.Current.Accent),
				ColorSequenceKeypoint.new(1, color2 or Theme.Current.Good),
			},
			Transparency = NumberSequence.new{
				NumberSequenceKeypoint.new(0,0),
				NumberSequenceKeypoint.new(1,1),
			},
			Lifetime     = 0.8,
			MinLength    = 0,
			Parent       = root,
		})
	end,
	Disable = function()
		State.TrailActive = false
		if _activeTrail then _activeTrail:Destroy(); _activeTrail=nil end
	end,
}
RegFeature("Trail Creator", "Visuals", "Leave a coloured trail behind your character")

local _activeAura
local AuraEffect = {
	Enable = function(color)
		State.AuraActive = true
		local root = HRP(); if not root then return end
		if _activeAura then _activeAura:Destroy() end
		_activeAura = Util.New("ParticleEmitter", {
			Color          = ColorSequence.new(color or Theme.Current.Accent),
			LightEmission  = 1,
			LightInfluence = 0,
			Size           = NumberSequence.new{NumberSequenceKeypoint.new(0,1.2),NumberSequenceKeypoint.new(1,0)},
			Transparency   = NumberSequence.new{NumberSequenceKeypoint.new(0,0),NumberSequenceKeypoint.new(1,1)},
			Lifetime       = NumberRange.new(0.6,1.2),
			Rate           = 40,
			Speed          = NumberRange.new(2,6),
			SpreadAngle    = Vector2.new(360,360),
			Parent         = root,
		})
	end,
	Disable = function()
		State.AuraActive = false
		if _activeAura then _activeAura:Destroy(); _activeAura=nil end
	end,
}
RegFeature("Custom Aura Effects", "Visuals", "Emit particles from your character")

local function createFloatingText(text, color)
	local root = HRP(); if not root then return end
	local bb = Util.New("BillboardGui", {
		Adornee     = root,
		Size        = UDim2.new(0,200,0,40),
		StudsOffset = Vector3.new(0,4,0),
		AlwaysOnTop = true,
		Parent      = root,
	})
	Util.New("TextLabel", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,0,1,0),
		Font                  = Enum.Font.GothamBold,
		Text                  = text,
		TextColor3            = color or Theme.Current.Accent,
		TextStrokeTransparency= 0,
		TextSize              = 18,
		Parent                = bb,
	})
	local t = 0
	local conn
	conn = RunService.Heartbeat:Connect(function(dt)
		t += dt
		if not root or not root.Parent then conn:Disconnect(); bb:Destroy(); return end
		bb.StudsOffset = Vector3.new(0, 4 + t*2, 0)
		if t >= 2.5 then conn:Disconnect(); bb:Destroy() end
	end)
end
RegFeature("Floating Text Creator", "Visuals", "Display floating text above your head")

----------------------------------------------------------------------------
-- ═══════════════════════════════════════════════════════════════════════
--  ESP SYSTEM — Drawing-API renderer ported from Blissful4992/ESPs
--  ▸ Box ESP      : Drawing.new("Quad") pair (colour + black border)
--                   Blissful 2D Box ESP / ESP+Health Bars scripts
--  ▸ Corner Box   : 8 Drawing.new("Line") L-brackets + rainbow cycling
--                   Blissful Corner 2D Box ESP script
--  ▸ Tracers      : Drawing.new("Line") from screen-bottom to target feet
--                   + black shadow line  (Blissful 2D Box ESP style)
--  ▸ Health Bars  : two Lines per target — black bg + lerped fill
--                   (Blissful 2D Box ESP / ESP+Health Bars)
--  ▸ Skeleton ESP : Drawing.new("Line") per limb, R15/R6 aware
--  ▸ Name/Dist/HP : Drawing.new("Text") labels above target
-- ═══════════════════════════════════════════════════════════════════════
----------------------------------------------------------------------------

local DRAWING_AVAILABLE = typeof(Drawing) == "table" or typeof(Drawing) == "userdata"

-- Shared colour constants (mirrors Blissful)
local _BLACK = Color3.fromRGB(0, 0, 0)
local _RED   = Color3.fromRGB(255, 0, 0)
local _GREEN = Color3.fromRGB(0, 255, 0)

------------------------------------------------------------------
-- Low-level Drawing constructors (one per shape type used)
------------------------------------------------------------------
local function NewLine(color, thickness)
	if not DRAWING_AVAILABLE then return nil end
	local l = Drawing.new("Line")
	l.Visible      = false
	l.From         = Vector2.new(0,0)
	l.To           = Vector2.new(0,0)
	l.Color        = color or Color3.new(1,1,1)
	l.Thickness    = thickness or 1
	l.Transparency = 1
	return l
end

-- Quad: four screen-space corner points — more accurate than Square.
-- Used for the full box (Blissful 2D Box ESP approach).
local function NewQuad(color, thickness)
	if not DRAWING_AVAILABLE then return nil end
	local q = Drawing.new("Quad")
	q.Visible      = false
	q.PointA       = Vector2.new(0,0)
	q.PointB       = Vector2.new(0,0)
	q.PointC       = Vector2.new(0,0)
	q.PointD       = Vector2.new(0,0)
	q.Color        = color or Color3.new(1,1,1)
	q.Filled       = false
	q.Thickness    = thickness or 1
	q.Transparency = 1
	return q
end

local function NewText(color, size, center)
	if not DRAWING_AVAILABLE then return nil end
	local t = Drawing.new("Text")
	t.Visible      = false
	t.Color        = color or Color3.new(1,1,1)
	t.Size         = size or 13
	t.Center       = center ~= false
	t.Outline      = true
	t.Font         = 2
	t.Text         = ""
	return t
end

------------------------------------------------------------------
-- Drawing lifecycle helpers
------------------------------------------------------------------
local function destroyDrawing(d)
	if d then pcall(function() d:Remove() end) end
end

-- Build the 8-line table for corner brackets (Blissful Corner 2D Box ESP)
local function newCornerLines(color, thickness)
	local lines = {}
	for i = 1, 8 do
		lines[i] = NewLine(color, thickness)
	end
	return lines
end

-- Build limb-line tables for skeleton (R15 or R6)
local R15_LIMBS = {
	{"Head_UpperTorso",             "Head",         "UpperTorso"},
	{"UpperTorso_LowerTorso",       "UpperTorso",   "LowerTorso"},
	{"UpperTorso_LeftUpperArm",     "UpperTorso",   "LeftUpperArm"},
	{"LeftUpperArm_LeftLowerArm",   "LeftUpperArm", "LeftLowerArm"},
	{"LeftLowerArm_LeftHand",       "LeftLowerArm", "LeftHand"},
	{"UpperTorso_RightUpperArm",    "UpperTorso",   "RightUpperArm"},
	{"RightUpperArm_RightLowerArm", "RightUpperArm","RightLowerArm"},
	{"RightLowerArm_RightHand",     "RightLowerArm","RightHand"},
	{"LowerTorso_LeftUpperLeg",     "LowerTorso",   "LeftUpperLeg"},
	{"LeftUpperLeg_LeftLowerLeg",   "LeftUpperLeg", "LeftLowerLeg"},
	{"LeftLowerLeg_LeftFoot",       "LeftLowerLeg", "LeftFoot"},
	{"LowerTorso_RightUpperLeg",    "LowerTorso",   "RightUpperLeg"},
	{"RightUpperLeg_RightLowerLeg", "RightUpperLeg","RightLowerLeg"},
	{"RightLowerLeg_RightFoot",     "RightLowerLeg","RightFoot"},
}

local function buildLimbDrawings(rigType, color)
	local limbs = {}
	if rigType == Enum.HumanoidRigType.R15 then
		for _, def in ipairs(R15_LIMBS) do
			limbs[def[1]] = NewLine(color)
		end
	else -- R6
		local names = {
			"Head_Spine","Spine",
			"LeftArm","LeftArm_UpperTorso",
			"RightArm","RightArm_UpperTorso",
			"LeftLeg","LeftLeg_LowerTorso",
			"RightLeg","RightLeg_LowerTorso",
		}
		for _, n in ipairs(names) do limbs[n] = NewLine(color) end
	end
	return limbs
end

------------------------------------------------------------------
-- Per-target ESP entry table
-- (created once, rendered in the heartbeat loop below)
------------------------------------------------------------------
local ESPHolders = {}

local function destroyESPData(data)
	if not data then return end
	-- Quad box
	destroyDrawing(data.box)
	destroyDrawing(data.blackBox)
	-- Tracers
	destroyDrawing(data.tracer)
	destroyDrawing(data.blackTracer)
	-- Health bars
	destroyDrawing(data.healthBar)
	destroyDrawing(data.healthBarBg)
	-- Text
	destroyDrawing(data.nameText)
	destroyDrawing(data.distText)
	destroyDrawing(data.hpText)
	-- Corner lines
	if data.corners then
		for _, l in ipairs(data.corners) do destroyDrawing(l) end
	end
	-- Skeleton limbs
	if data.limbs then
		for _, l in pairs(data.limbs) do destroyDrawing(l) end
	end
end

local function clearESP(category)
	if not DRAWING_AVAILABLE then return end
	for inst, data in pairs(ESPHolders) do
		if data.category == category then
			destroyESPData(data)
			ESPHolders[inst] = nil
		end
	end
end

local function clearAllESP()
	if not DRAWING_AVAILABLE then return end
	for _, data in pairs(ESPHolders) do destroyESPData(data) end
	ESPHolders = {}
end

------------------------------------------------------------------
-- Helper: team colour
------------------------------------------------------------------
local function getTeamColor(plr)
	if plr and plr.Team then return plr.Team.TeamColor.Color end
	return Theme.Current.Accent
end

------------------------------------------------------------------
-- Helper: Blissful health colour lerp (red → green)
------------------------------------------------------------------
local function healthColor(ratio)
	return _RED:Lerp(_GREEN, ratio)
end

------------------------------------------------------------------
-- Blissful box corner math
-- Projects a quad from rootPart CFrame using the head-to-hip
-- DistanceY as the scale factor, exactly as in the Blissful
-- 2D Box ESP and Corner 2D Box ESP scripts.
------------------------------------------------------------------
local function getBlissfulBoxPoints(rootPos, humPos2d, distY)
	-- Returns screen-space V2 for TL, TR, BL, BR
	local TL = Camera:WorldToViewportPoint(
		(CFrame.new(rootPos) * CFrame.new( distY, distY*2, 0)).Position)
	local TR = Camera:WorldToViewportPoint(
		(CFrame.new(rootPos) * CFrame.new(-distY, distY*2, 0)).Position)
	local BL = Camera:WorldToViewportPoint(
		(CFrame.new(rootPos) * CFrame.new( distY,-distY*2, 0)).Position)
	local BR = Camera:WorldToViewportPoint(
		(CFrame.new(rootPos) * CFrame.new(-distY,-distY*2, 0)).Position)
	return Vector2.new(TL.X,TL.Y), Vector2.new(TR.X,TR.Y),
	       Vector2.new(BL.X,BL.Y), Vector2.new(BR.X,BR.Y)
end

------------------------------------------------------------------
-- Corner box helper — places 8 line segments (Blissful style)
-- lines[1..8] = TL_H, TL_V, TR_H, TR_V, BL_H, BL_V, BR_H, BR_V
------------------------------------------------------------------
local function updateCornerLines(lines, TL, TR, BL, BR, offset, color, thickness)
	if not lines then return end
	-- TL corner
	lines[1].From = TL;                    lines[1].To = Vector2.new(TL.X+offset, TL.Y)
	lines[2].From = TL;                    lines[2].To = Vector2.new(TL.X, TL.Y+offset)
	-- TR corner
	lines[3].From = TR;                    lines[3].To = Vector2.new(TR.X-offset, TR.Y)
	lines[4].From = TR;                    lines[4].To = Vector2.new(TR.X, TR.Y+offset)
	-- BL corner
	lines[5].From = BL;                    lines[5].To = Vector2.new(BL.X+offset, BL.Y)
	lines[6].From = BL;                    lines[6].To = Vector2.new(BL.X, BL.Y-offset)
	-- BR corner
	lines[7].From = BR;                    lines[7].To = Vector2.new(BR.X-offset, BR.Y)
	lines[8].From = BR;                    lines[8].To = Vector2.new(BR.X, BR.Y-offset)
	for _, l in ipairs(lines) do
		l.Color     = color
		l.Thickness = thickness
		l.Visible   = true
	end
end

------------------------------------------------------------------
-- Rainbow state for corner boxes (hue cycles per-frame)
------------------------------------------------------------------
local _rainbowHue = 0
RunService.Heartbeat:Connect(function(dt)
	_rainbowHue = (_rainbowHue + dt * 60) % 360
end)

------------------------------------------------------------------
-- addESP: creates all drawing objects for one target
------------------------------------------------------------------
local function addESP(inst, category, colorFn, labelFn, plr, getCharacter)
	if not DRAWING_AVAILABLE then return end
	if ESPHolders[inst] then return end

	local baseColor = (colorFn and colorFn()) or Theme.Current.Accent

	local rigType = nil
	if getCharacter then
		local char = getCharacter()
		local hum  = char and char:FindFirstChildOfClass("Humanoid")
		if hum then rigType = hum.RigType end
	end

	ESPHolders[inst] = {
		category     = category,
		colorFn      = colorFn,
		labelFn      = labelFn,
		plr          = plr,
		getCharacter = getCharacter,
		rigType      = rigType,
		-- Full box (Blissful 2D Box ESP — Quad pair)
		box          = NewQuad(baseColor, 1),
		blackBox     = NewQuad(_BLACK, 2),        -- drawn first (behind)
		-- Line tracers (Blissful style: screen-bottom → target feet)
		tracer       = NewLine(baseColor, 1),
		blackTracer  = NewLine(_BLACK, 2),        -- drawn first (behind)
		-- Health bars (Blissful: two Lines on the left edge of the box)
		healthBar    = NewLine(_GREEN, 1.5),
		healthBarBg  = NewLine(_BLACK, 3),
		-- Text overlays
		nameText     = NewText(baseColor, 14),
		distText     = NewText(Color3.new(1,1,1), 12),
		hpText       = NewText(Color3.fromRGB(80,255,100), 12),
		-- Corner lines (Blissful Corner 2D Box ESP — 8 Lines)
		corners      = newCornerLines(baseColor, 1),
		-- Skeleton limbs (built lazily)
		limbs        = nil,
	}
end

------------------------------------------------------------------
-- ESP main render loop (Heartbeat)
------------------------------------------------------------------
local espHeartbeat
local function startESPLoop()
	if not DRAWING_AVAILABLE then return end
	if espHeartbeat then return end

	espHeartbeat = RunService.Heartbeat:Connect(function()
		local myRoot = HRP()

		for inst, data in pairs(ESPHolders) do
			-- Clean up stale entries
			if not inst or not inst.Parent then
				destroyESPData(data)
				ESPHolders[inst] = nil
			else
				local char  = (data.getCharacter and data.getCharacter()) or inst:FindFirstAncestorOfClass("Model")
				local hum   = char and char:FindFirstChildOfClass("Humanoid")
				local alive = (not hum) or hum.Health > 0

				-- Hide everything if dead/invalid
				if not alive then
					for _, k in ipairs({"box","blackBox","tracer","blackTracer",
					                    "healthBar","healthBarBg","nameText","distText","hpText"}) do
						if data[k] then data[k].Visible = false end
					end
					if data.corners then for _,l in ipairs(data.corners) do l.Visible=false end end
					if data.limbs   then for _,l in pairs(data.limbs)   do l.Visible=false end end
				else
					-- ── Resolve current colour ──────────────────────────────────
					local baseColor = (data.colorFn and data.colorFn()) or Theme.Current.Accent
					if State.ESP.TeamColor and data.plr then
						baseColor = getTeamColor(data.plr)
					end
					-- (Health colour is applied per-drawing below)

					-- ── Screen projection ────────────────────────────────────────
					local rootPos   = inst.Position
					local humPos3, onScreen = Camera:WorldToViewportPoint(rootPos)
					local humPos2  = Vector2.new(humPos3.X, humPos3.Y)

					-- Head position for DistanceY scaling (Blissful approach)
					local head     = char and char:FindFirstChild("Head")
					local headPos3 = head and Camera:WorldToViewportPoint(head.Position)
					local distY    = 0
					if headPos3 then
						distY = math.clamp(
							(Vector2.new(headPos3.X, headPos3.Y) - humPos2).Magnitude,
							2, math.huge)
					end

					-- Distance to local player (in studs)
					local dist = myRoot and math.floor((myRoot.Position - rootPos).Magnitude) or 0

					-- Box corner screen positions (shared by box and corner modes)
					local TL, TR, BL, BR
					if onScreen and distY > 0 then
						TL, TR, BL, BR = getBlissfulBoxPoints(rootPos, humPos2, distY)
					end

					-- ── Full Quad Box ────────────────────────────────────────────
					-- Drawn only when Box mode on AND not replaced by Skeleton
					if State.ESP.Boxes and not State.ESP.Skeleton and onScreen and TL then
						-- Black border quad first (it's "behind" the coloured one visually
						-- because Drawing renders in insertion order — but since we can't
						-- guarantee that here, we make the black quad slightly thicker)
						if data.blackBox then
							data.blackBox.PointA  = TL
							data.blackBox.PointB  = TR
							data.blackBox.PointC  = BR
							data.blackBox.PointD  = BL
							data.blackBox.Visible = true
						end
						if data.box then
							data.box.PointA  = TL
							data.box.PointB  = TR
							data.box.PointC  = BR
							data.box.PointD  = BL
							data.box.Color   = baseColor
							data.box.Visible = true
						end
					else
						if data.box     then data.box.Visible     = false end
						if data.blackBox then data.blackBox.Visible = false end
					end

					-- ── Corner Box ───────────────────────────────────────────────
					-- (Blissful Corner 2D Box ESP — rainbow cycling + auto-thickness)
					if State.ESP.CornerBox and onScreen and TL and data.corners then
						local ratio    = dist > 0 and 1/dist or 0
						local offset   = math.clamp(ratio * 750, 2, 300)
						-- auto-thickness scaled with distance (Blissful feature)
						local thick    = math.clamp(ratio * 100, 1, 4)
						-- rainbow colour cycle
						local rainbowC = Color3.fromHSV(_rainbowHue/360, 0.6, 1)
						updateCornerLines(data.corners, TL, TR, BL, BR, offset, rainbowC, thick)
					else
						if data.corners then
							for _, l in ipairs(data.corners) do l.Visible=false end
						end
					end

					-- ── Skeleton ─────────────────────────────────────────────────
					if State.ESP.Skeleton and onScreen and char then
						if not data.limbs and data.rigType then
							data.limbs = buildLimbDrawings(data.rigType, baseColor)
						end
						if data.limbs then
							if data.rigType == Enum.HumanoidRigType.R15 then
								for _, def in ipairs(R15_LIMBS) do
									local line     = data.limbs[def[1]]
									local fromPart = char:FindFirstChild(def[2])
									local toPart   = char:FindFirstChild(def[3])
									if line and fromPart and toPart then
										local A = Camera:WorldToViewportPoint(fromPart.Position)
										local B = Camera:WorldToViewportPoint(toPart.Position)
										line.From    = Vector2.new(A.X, A.Y)
										line.To      = Vector2.new(B.X, B.Y)
										line.Color   = baseColor
										line.Visible = true
									end
								end
							else -- R6
								local torso = char:FindFirstChild("Torso")
								local headP = char:FindFirstChild("Head")
								local lArm  = char:FindFirstChild("Left Arm")
								local rArm  = char:FindFirstChild("Right Arm")
								local lLeg  = char:FindFirstChild("Left Leg")
								local rLeg  = char:FindFirstChild("Right Leg")
								if torso and headP and lArm and rArm and lLeg and rLeg then
									local tH  = torso.Size.Y/2 - 0.2
									local UT  = Camera:WorldToViewportPoint((torso.CFrame * CFrame.new(0,tH,0)).Position)
									local LT  = Camera:WorldToViewportPoint((torso.CFrame * CFrame.new(0,-tH,0)).Position)
									local H   = Camera:WorldToViewportPoint(headP.Position)
									local laH = lArm.Size.Y/2 - 0.2
									local LUA = Camera:WorldToViewportPoint((lArm.CFrame * CFrame.new(0,laH,0)).Position)
									local LLA = Camera:WorldToViewportPoint((lArm.CFrame * CFrame.new(0,-laH,0)).Position)
									local raH = rArm.Size.Y/2 - 0.2
									local RUA = Camera:WorldToViewportPoint((rArm.CFrame * CFrame.new(0,raH,0)).Position)
									local RLA = Camera:WorldToViewportPoint((rArm.CFrame * CFrame.new(0,-raH,0)).Position)
									local llH = lLeg.Size.Y/2 - 0.2
									local LUL = Camera:WorldToViewportPoint((lLeg.CFrame * CFrame.new(0,llH,0)).Position)
									local LLL = Camera:WorldToViewportPoint((lLeg.CFrame * CFrame.new(0,-llH,0)).Position)
									local rlH = rLeg.Size.Y/2 - 0.2
									local RUL = Camera:WorldToViewportPoint((rLeg.CFrame * CFrame.new(0,rlH,0)).Position)
									local RLL = Camera:WorldToViewportPoint((rLeg.CFrame * CFrame.new(0,-rlH,0)).Position)
									local L   = data.limbs
									L.Head_Spine.From            = Vector2.new(H.X,H.Y)
									L.Head_Spine.To              = Vector2.new(UT.X,UT.Y)
									L.Spine.From                 = Vector2.new(UT.X,UT.Y)
									L.Spine.To                   = Vector2.new(LT.X,LT.Y)
									L.LeftArm.From               = Vector2.new(LUA.X,LUA.Y)
									L.LeftArm.To                 = Vector2.new(LLA.X,LLA.Y)
									L.LeftArm_UpperTorso.From    = Vector2.new(UT.X,UT.Y)
									L.LeftArm_UpperTorso.To      = Vector2.new(LUA.X,LUA.Y)
									L.RightArm.From              = Vector2.new(RUA.X,RUA.Y)
									L.RightArm.To                = Vector2.new(RLA.X,RLA.Y)
									L.RightArm_UpperTorso.From   = Vector2.new(UT.X,UT.Y)
									L.RightArm_UpperTorso.To     = Vector2.new(RUA.X,RUA.Y)
									L.LeftLeg.From               = Vector2.new(LUL.X,LUL.Y)
									L.LeftLeg.To                 = Vector2.new(LLL.X,LLL.Y)
									L.LeftLeg_LowerTorso.From    = Vector2.new(LT.X,LT.Y)
									L.LeftLeg_LowerTorso.To      = Vector2.new(LUL.X,LUL.Y)
									L.RightLeg.From              = Vector2.new(RUL.X,RUL.Y)
									L.RightLeg.To                = Vector2.new(RLL.X,RLL.Y)
									L.RightLeg_LowerTorso.From   = Vector2.new(LT.X,LT.Y)
									L.RightLeg_LowerTorso.To     = Vector2.new(RUL.X,RUL.Y)
									for _, l in pairs(L) do l.Color=baseColor; l.Visible=true end
								end
							end
						end
					else
						if data.limbs then for _,l in pairs(data.limbs) do l.Visible=false end end
					end

					-- ── Tracers (Blissful style: screen-bottom line to target feet) ──
					-- Drawn for ALL targets whether on-screen or off-screen.
					if State.ESP.Tracers then
						local screenBottom = Vector2.new(Camera.ViewportSize.X * 0.5, Camera.ViewportSize.Y)
						-- "feet" = bottom of the box on screen
						local feetY  = BL and BL.Y or humPos2.Y + distY*2
						local feetX  = humPos2.X
						local targetV= Vector2.new(feetX, feetY)

						if data.blackTracer then
							data.blackTracer.From    = screenBottom
							data.blackTracer.To      = targetV
							data.blackTracer.Visible = true
						end
						if data.tracer then
							data.tracer.From    = screenBottom
							data.tracer.To      = targetV
							data.tracer.Color   = baseColor
							data.tracer.Visible = true
						end
					else
						if data.tracer      then data.tracer.Visible      = false end
						if data.blackTracer then data.blackTracer.Visible = false end
					end

					-- ── Health Bars (Blissful 2D Box ESP style) ─────────────────
					-- Left edge of the box, spanning full box height.
					-- Black background line first, colour-lerped fill on top.
					if State.ESP.HealthBars and onScreen and hum and BL then
						local ratio       = hum.Health / math.max(hum.MaxHealth, 1)
						local barX        = BL.X - 4
						local barBottom   = BL.Y      -- bottom-left Y
						local barTop      = TL.Y      -- top-left Y
						local fullHeight2 = (Vector2.new(barX, barTop) - Vector2.new(barX, barBottom)).Magnitude
						local fillHeight  = ratio * fullHeight2

						if data.healthBarBg then
							data.healthBarBg.From    = Vector2.new(barX, barBottom)
							data.healthBarBg.To      = Vector2.new(barX, barTop)
							data.healthBarBg.Visible = true
						end
						if data.healthBar then
							data.healthBar.From    = Vector2.new(barX, barBottom)
							data.healthBar.To      = Vector2.new(barX, barBottom - fillHeight)
							data.healthBar.Color   = healthColor(ratio)
							data.healthBar.Visible = true
						end
					else
						if data.healthBar   then data.healthBar.Visible   = false end
						if data.healthBarBg then data.healthBarBg.Visible = false end
					end

					-- ── Name / Distance / Health text ────────────────────────────
					if onScreen then
						local textY = humPos2.Y - distY*2 - 4
						if State.ESP.Names then
							if data.nameText then
								data.nameText.Text     = (data.labelFn and data.labelFn()) or inst.Name
								data.nameText.Color    = baseColor
								data.nameText.Position = Vector2.new(humPos2.X, textY - 16)
								data.nameText.Visible  = true
							end
						else
							if data.nameText then data.nameText.Visible = false end
						end
						if State.ESP.Distances then
							if data.distText then
								data.distText.Text     = dist.."m"
								data.distText.Position = Vector2.new(humPos2.X, textY - (State.ESP.Names and 30 or 14))
								data.distText.Visible  = true
							end
						else
							if data.distText then data.distText.Visible = false end
						end
						if State.ESP.Health and hum then
							local ratio = hum.Health / math.max(hum.MaxHealth,1)
							if data.hpText then
								data.hpText.Text     = "HP: "..math.floor(ratio*100).."%"
								data.hpText.Color    = healthColor(ratio)
								data.hpText.Position = Vector2.new(humPos2.X, textY - (State.ESP.Names and 44 or 28))
								data.hpText.Visible  = true
							end
						else
							if data.hpText then data.hpText.Visible = false end
						end
					else
						if data.nameText then data.nameText.Visible = false end
						if data.distText then data.distText.Visible = false end
						if data.hpText   then data.hpText.Visible   = false end
					end
				end -- alive
			end -- inst.Parent
		end -- for ESPHolders
	end) -- Heartbeat
end

------------------------------------------------------------------
-- World scan — registers ESP entries for active categories
------------------------------------------------------------------
local function rescanWorld()
	if not DRAWING_AVAILABLE then return end

	if State.ESP.Players then
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LocalPlayer and plr.Character then
				local root = plr.Character:FindFirstChild("HumanoidRootPart")
				if root and not ESPHolders[root] then
					addESP(root, "Players",
						function()
							if State.ESP.TeamColor then return getTeamColor(plr) end
							return Theme.Current.Accent
						end,
						function() return plr.DisplayName end,
						plr,
						function() return plr.Character end)
				end
			end
		end
	else clearESP("Players") end

	if State.ESP.NPCs then
		for _, desc in ipairs(Workspace:GetDescendants()) do
			if desc:IsA("Humanoid") then
				local model = desc.Parent
				local root  = model and model:FindFirstChild("HumanoidRootPart")
				if root and not Players:GetPlayerFromCharacter(model) and not ESPHolders[root] then
					addESP(root, "NPCs",
						function() return Theme.Current.Bad end,
						function() return model.Name end,
						nil, function() return model end)
				end
			end
		end
	else clearESP("NPCs") end

	if State.ESP.Items then
		for _, inst in ipairs(CollectionService:GetTagged("ESP_Item")) do
			local target = inst:IsA("BasePart") and inst or inst:FindFirstChildWhichIsA("BasePart",true)
			if target and not ESPHolders[target] then
				addESP(target, "Items",
					function() return Theme.Current.Good end,
					function() return inst.Name end)
			end
		end
	else clearESP("Items") end

	if State.ESP.Objectives then
		for _, inst in ipairs(CollectionService:GetTagged("ESP_Objective")) do
			local target = inst:IsA("BasePart") and inst or inst:FindFirstChildWhichIsA("BasePart",true)
			if target and not ESPHolders[target] then
				addESP(target, "Objectives",
					function() return Color3.fromRGB(255,200,60) end,
					function() return inst.Name end)
			end
		end
	else clearESP("Objectives") end

	if State.ESP.Vehicles then
		for _, model in ipairs(Workspace:GetDescendants()) do
			if model:IsA("Model") and model:FindFirstChildOfClass("VehicleSeat") then
				local root = model:FindFirstChildWhichIsA("BasePart", true)
				if root and not ESPHolders[root] then
					addESP(root, "Vehicles",
						function() return Color3.fromRGB(255,180,0) end,
						function() return model.Name end)
				end
			end
		end
	else clearESP("Vehicles") end
end

local function refreshESP()
	if not DRAWING_AVAILABLE then
		Notify.Push("ESP Unavailable",
			"Drawing API requires an executor environment", "Warn", 3)
		return
	end
	rescanWorld()
	startESPLoop()
end

-- Periodic re-scan for new players/NPCs joining
task.spawn(function()
	while task.wait(2) do
		local any = false
		for _, v in pairs(State.ESP) do if v then any=true; break end end
		if any then rescanWorld() end
	end
end)

-- All-categories rebuild helper used by every toggle
local function espToggleRebuild()
	clearAllESP()
	refreshESP()
end

RegFeature("ESP Players",        "Visuals", "See player positions (Blissful Quad box)")
RegFeature("ESP NPCs",           "Visuals", "See NPC positions (Blissful Quad box)")
RegFeature("ESP Items",          "Visuals", "Highlight items tagged ESP_Item")
RegFeature("ESP Objectives",     "Visuals", "Highlight objectives tagged ESP_Objective")
RegFeature("Box ESP",            "Visuals", "Blissful 2D Quad box (colour + black border)")
RegFeature("Corner Box ESP",     "Visuals", "Blissful L-bracket corners — rainbow & auto-thickness")
RegFeature("Tracer ESP",         "Visuals", "Screen-bottom line to target feet (Blissful style)")
RegFeature("Health Bars (drawn)","Visuals", "Blissful-style HP bar on left edge of box")
RegFeature("Name ESP",           "Visuals", "Show name labels above targets")
RegFeature("Distance ESP",       "Visuals", "Show distance in studs")
RegFeature("Health Text ESP",    "Visuals", "Show HP % as text label")
RegFeature("Skeleton ESP",       "Visuals", "Full limb skeleton R15/R6 — replaces Box when active")
RegFeature("Team-Color ESP",     "Visuals", "Colour targets by their team colour")
RegFeature("Vehicle ESP",        "Visuals", "Highlight vehicles in the world")

----------------------------------------------------------------------------
-- FEATURE: X-Ray Mode
----------------------------------------------------------------------------
local XRay = {
	Enable = function()
		State.XRay = true
		local h = Util.New("Highlight", {
			Name                = "XRayHL",
			FillTransparency    = 1,
			OutlineTransparency = 0.5,
			OutlineColor        = Color3.fromRGB(100,200,255),
			DepthMode           = Enum.HighlightDepthMode.AlwaysOnTop,
			Parent              = Workspace,
		})
	end,
	Disable = function()
		State.XRay = false
		local h = Workspace:FindFirstChild("XRayHL")
		if h then h:Destroy() end
	end,
}
RegFeature("X-Ray Mode", "Visuals", "See through world geometry with highlight outlines")

----------------------------------------------------------------------------
-- FEATURE: Highlight Interactables / Remove Clutter
----------------------------------------------------------------------------
local _interactHL = {}
local HighlightInteractables = {
	Enable = function()
		for _, inst in ipairs(CollectionService:GetTagged("Interactable")) do
			local target = inst:IsA("BasePart") and inst or inst:FindFirstChildWhichIsA("BasePart",true)
			if target then
				local h = Util.New("Highlight", {
					Adornee          = target,
					FillColor        = Color3.fromRGB(255,240,100),
					FillTransparency = 0.6,
					OutlineColor     = Color3.fromRGB(255,240,100),
					Parent           = target,
				})
				table.insert(_interactHL, h)
			end
		end
	end,
	Disable = function()
		for _, h in ipairs(_interactHL) do pcall(function() h:Destroy() end) end
		_interactHL = {}
	end,
}
RegFeature("Highlight Interactables", "Visuals", "Glow items tagged 'Interactable'")

local _savedDecorTrans = {}
local RemoveClutter = {
	Enable = function()
		for _, inst in ipairs(Workspace:GetDescendants()) do
			if inst:IsA("Decal") or inst:IsA("Texture") or inst:IsA("ParticleEmitter") then
				_savedDecorTrans[inst] = inst:IsA("ParticleEmitter") and inst.Rate or inst.Transparency
				if inst:IsA("ParticleEmitter") then inst.Rate=0 else inst.Transparency=1 end
			end
		end
	end,
	Disable = function()
		for inst, v in pairs(_savedDecorTrans) do
			if inst and inst.Parent then
				if inst:IsA("ParticleEmitter") then inst.Rate=v else inst.Transparency=v end
			end
		end
		_savedDecorTrans = {}
	end,
}
RegFeature("Remove Visual Clutter", "Visuals", "Hide decals, textures, and particles")

----------------------------------------------------------------------------
-- WORLD FEATURES
----------------------------------------------------------------------------
RegFeature("Time of Day",      "World", "Set the in-game clock time")
RegFeature("Weather Changer",  "World", "Toggle rain/storm/clear sky")
RegFeature("Fog Controls",     "World", "Change fog start/end distance")
RegFeature("Brightness Controls","World","Adjust ambient lighting brightness")
RegFeature("Color Correction", "World", "Apply color filter post-processing")

local ColorCorrection = Instance.new("ColorCorrectionEffect")
ColorCorrection.Parent  = Lighting
ColorCorrection.Enabled = false

local SunRaysEffect = Instance.new("SunRaysEffect")
SunRaysEffect.Parent  = Lighting
SunRaysEffect.Enabled = false

local function setTime(v) Lighting.ClockTime = v end

local weatherPresets = {
	Clear  = { FogEnd=1e8,  FogColor=Color3.fromRGB(200,220,240), Brightness=2   },
	Cloudy = { FogEnd=4000, FogColor=Color3.fromRGB(160,170,180), Brightness=0.5 },
	Rainy  = { FogEnd=800,  FogColor=Color3.fromRGB(130,140,150), Brightness=0.3 },
	Storm  = { FogEnd=400,  FogColor=Color3.fromRGB(80,85,90),    Brightness=0.2 },
	Foggy  = { FogEnd=200,  FogColor=Color3.fromRGB(200,200,205), Brightness=1   },
}
local function setWeather(name)
	local p = weatherPresets[name]; if not p then return end
	Lighting.FogEnd    = p.FogEnd
	Lighting.FogColor  = p.FogColor
	Lighting.Brightness= p.Brightness
	Notify.Push("Weather", name, "Info", 2)
end

local function doLightningStrike()
	local root = HRP(); if not root then return end
	local pos  = root.Position + Vector3.new(math.random(-20,20),0,math.random(-20,20))
	local savedBright = Lighting.Brightness
	Lighting.Brightness = 10
	local bolt = Util.New("Part", {
		Name=         "LightningBolt", Anchored=true, CanCollide=false,
		Size=         Vector3.new(0.3,60,0.3),
		CFrame=       CFrame.new(pos + Vector3.new(0,30,0)),
		Material=     Enum.Material.Neon,
		Color=        Color3.fromRGB(180,180,255),
		Parent=       Workspace,
	})
	startCameraShake(Settings.CamShakeAmt*2, 0.6)
	task.delay(0.08, function() Lighting.Brightness=savedBright end)
	task.delay(0.4,  function() if bolt and bolt.Parent then bolt:Destroy() end end)
	Notify.Push("Lightning Strike", "⚡ ZAP!", "Info", 1.5)
end

local function doMeteorEffect()
	local root = HRP(); if not root then return end
	local target = root.Position + Vector3.new(math.random(-15,15),0,math.random(-15,15))
	local meteor = Util.New("Part", {
		Name=    "Meteor", Anchored=false, CanCollide=true,
		Size=    Vector3.new(4,4,4), Shape=Enum.PartType.Ball,
		Material=Enum.Material.Neon, Color=Color3.fromRGB(255,120,20),
		CFrame=  CFrame.new(target + Vector3.new(0,80,0)), Parent=Workspace,
	})
	Util.New("ParticleEmitter", {
		Color=ColorSequence.new(Color3.fromRGB(255,60,0)),
		Size=NumberSequence.new(2), Rate=100, Speed=NumberRange.new(2,6), Parent=meteor,
	})
	meteor:ApplyImpulse(Vector3.new(0,-500,0))
	task.delay(3, function() if meteor and meteor.Parent then meteor:Destroy() end end)
end

local _eqConn
local function doEarthquake(duration)
	if _eqConn then _eqConn:Disconnect() end
	local t = 0
	_eqConn = RunService.RenderStepped:Connect(function(dt)
		t += dt
		if t >= (duration or 3) then _eqConn:Disconnect(); _eqConn=nil; return end
		Camera.CFrame = Camera.CFrame * CFrame.new(math.sin(t*40)*Settings.CamShakeAmt*0.5, 0, 0)
	end)
end

local _selfParticle
local function spawnSelfParticles(color)
	local root = HRP(); if not root then return end
	if _selfParticle then _selfParticle:Destroy() end
	_selfParticle = Util.New("ParticleEmitter", {
		Color=        ColorSequence.new(color or Color3.fromRGB(255,200,50)),
		LightEmission=0.5,
		Size=         NumberSequence.new{NumberSequenceKeypoint.new(0,1.5),NumberSequenceKeypoint.new(1,0)},
		Transparency= NumberSequence.new{NumberSequenceKeypoint.new(0,0),NumberSequenceKeypoint.new(1,1)},
		Lifetime=     NumberRange.new(0.5,1.5),
		Rate=         30, Speed=NumberRange.new(5,15),
		SpreadAngle=  Vector2.new(180,180),
		Parent=       root,
	})
end

local function teleportTo(targetPlayer)
	local root       = HRP()
	local targetChar = targetPlayer.Character
	local targetRoot = targetChar and targetChar:FindFirstChild("HumanoidRootPart")
	if not root or not targetRoot then Notify.Push("Teleport Failed","Missing character data","Error"); return end
	State._lastTeleportCF = root.CFrame
	root.CFrame = targetRoot.CFrame * CFrame.new(0,0,4)
	Notify.Push("Teleported","Moved to "..targetPlayer.Name,"Success",2)
end

local Spectate = {
	Start = function(targetPlayer)
		local char = targetPlayer.Character
		local hum  = char and char:FindFirstChildOfClass("Humanoid")
		if not hum then Notify.Push("Spectate Failed",targetPlayer.Name.." has no character","Error"); return end
		State.Spectating     = targetPlayer
		Camera.CameraType    = Enum.CameraType.Custom
		Camera.CameraSubject = hum
		Notify.Push("Spectating",targetPlayer.Name.." — click Unspectate to stop","Info",3)
	end,
	Stop = function()
		State.Spectating    = nil
		local hum           = Humanoid()
		Camera.CameraSubject= hum or Camera.CameraSubject
		Camera.CameraType   = Enum.CameraType.Custom
		Notify.Push("Spectate","Stopped spectating","Info",2)
	end,
}

----------------------------------------------------------------------------
-- DEVELOPER FEATURES
----------------------------------------------------------------------------
local _remoteLog = {}
local RemoteLogger = {
	Enable = function(logBox)
		State.RemoteLogging = true
		local function hookRemote(inst)
			if inst:IsA("RemoteEvent") then
				inst.OnClientEvent:Connect(function(...)
					if not State.RemoteLogging then return end
					local args = {...}
					local msg  = "[RE] "..inst.Name.." → "
					for i, v in ipairs(args) do
						msg = msg..tostring(v)
						if i < #args then msg=msg..", " end
					end
					if logBox then logBox.Append(msg, Theme.Current.Good) end
					table.insert(_remoteLog, 1, {type="RE",name=inst.Name,time=os.time()})
					if #_remoteLog > 200 then table.remove(_remoteLog) end
				end)
			end
		end
		for _, inst in ipairs(game:GetDescendants()) do hookRemote(inst) end
		game.DescendantAdded:Connect(hookRemote)
	end,
	Disable = function() State.RemoteLogging=false end,
}
RegFeature("RemoteEvent Logger","Developer","Log all incoming RemoteEvent fires")
RegFeature("Instance Explorer", "Developer","Browse the DataModel tree")

local _errorLog = {}
pcall(function()
	game:GetService("ScriptContext").Error:Connect(function(msg, trace, script)
		table.insert(_errorLog, 1, "[ERR]"..(script and script.Name or "?")..": "..tostring(msg))
		if #_errorLog > 100 then table.remove(_errorLog) end
	end)
end)

local _colVizHL = {}
local CollisionViz = {
	Enable = function()
		State.ColVis = true
		for _, inst in ipairs(Workspace:GetDescendants()) do
			if inst:IsA("BasePart") and inst.CanCollide then
				local h = Util.New("SelectionBox", {
					Adornee=inst, Color3=Color3.fromRGB(0,200,255),
					LineThickness=0.04, SurfaceTransparency=0.9,
					SurfaceColor3=Color3.fromRGB(0,200,255), Parent=Workspace,
				})
				table.insert(_colVizHL, h)
			end
		end
	end,
	Disable = function()
		State.ColVis = false
		for _, h in ipairs(_colVizHL) do pcall(function() h:Destroy() end) end
		_colVizHL = {}
	end,
}

local _hitboxHL = {}
local HitboxViz = {
	Enable = function()
		State.HitVis = true
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr.Character then
				for _, part in ipairs(plr.Character:GetDescendants()) do
					if part:IsA("BasePart") then
						local h = Util.New("SelectionBox", {
							Adornee=part, Color3=Color3.fromRGB(255,50,50),
							LineThickness=0.05, SurfaceTransparency=0.85,
							SurfaceColor3=Color3.fromRGB(255,50,50), Parent=Workspace,
						})
						table.insert(_hitboxHL, h)
					end
				end
			end
		end
	end,
	Disable = function()
		State.HitVis = false
		for _, h in ipairs(_hitboxHL) do pcall(function() h:Destroy() end) end
		_hitboxHL = {}
	end,
}

local _lagConn
local LagSim = {
	Enable = function(ms)
		State.SimulateLag = true
		if _lagConn then _lagConn:Disconnect() end
		_lagConn = RunService.Heartbeat:Connect(function()
			if not State.SimulateLag then return end
			local wait_ms = (ms or 200)/1000
			local t = tick()
			while tick()-t < wait_ms do end
		end)
	end,
	Disable = function()
		State.SimulateLag = false
		if _lagConn then _lagConn:Disconnect(); _lagConn=nil end
	end,
}
RegFeature("Simulate Lag",        "Developer","Block Heartbeat thread to add input delay")
RegFeature("Hitbox Visualizer",   "Developer","Show player hitboxes as red boxes")
RegFeature("Collision Visualizer","Developer","Show collidable parts as blue boxes")

local _inspectorConn
local _inspectorActive = false
local function startInspector(logBox)
	_inspectorActive = true
	if _inspectorConn then _inspectorConn:Disconnect() end
	_inspectorConn = Mouse.Button1Down:Connect(function()
		if not _inspectorActive then return end
		local target = Mouse.Target; if not target then return end
		if logBox then
			logBox.Clear()
			logBox.Append("=== "..target:GetFullName().." ===", Theme.Current.Accent)
			logBox.Append("ClassName: "..target.ClassName, Theme.Current.Text)
			for _, prop in ipairs({"Name","Position","Size","Color","Material","Anchored","CanCollide","Transparency"}) do
				local ok, v = pcall(function() return tostring(target[prop]) end)
				if ok then logBox.Append(prop..": "..v, Theme.Current.SubText) end
			end
		end
	end)
end
local function stopInspector()
	_inspectorActive = false
	if _inspectorConn then _inspectorConn:Disconnect(); _inspectorConn=nil end
end

----------------------------------------------------------------------------
-- BUILD TABS
----------------------------------------------------------------------------
CreateTab("Movement",  "⚡", 1)
CreateTab("Visuals",   "👁", 2)
CreateTab("World",     "🌍", 3)
CreateTab("Players",   "👤", 4)
CreateTab("Developer", "🔧", 5)
CreateTab("QoL",       "⭐", 6)
CreateTab("Keybinds",  "⌨", 7)
CreateTab("Settings",  "⚙", 8)

-- =========================================================
-- MOVEMENT PAGE
-- =========================================================
do
	local page = Pages.Movement.page

	local s1 = Components.Section(page, "✈ Fly", 1)
	Components.Toggle(s1,"Enable Fly",false,function(v) if v then Fly.Enable() else Fly.Disable() end end,1,"WASD to move, Space=up, Shift=down")
	Components.Slider(s1,"Fly Speed",10,300,Settings.FlySpeed,function(v) Settings.FlySpeed=v; saveSettings() end,2)

	local s2 = Components.Section(page, "🏃 Movement", 2)
	Components.Toggle(s2,"Noclip",false,function(v) if v then Noclip.Enable() else Noclip.Disable() end end,1,"Walk through walls and parts")
	Components.Toggle(s2,"Infinite Jump",false,function(v) if v then InfJump.Enable() else InfJump.Disable() end end,2)
	Components.Toggle(s2,"Double Jump",false,function(v) if v then DoubleJump.Enable() else DoubleJump.Disable() end end,3)
	Components.Toggle(s2,"Sprint Toggle",false,function(v) Sprint.Toggle(v) end,4)
	Components.Toggle(s2,"Wall Climb",false,function(v) if v then WallClimb.Enable() else WallClimb.Disable() end end,5)
	Components.Toggle(s2,"Swim in Air",false,function(v) if v then SwimInAir.Enable() else SwimInAir.Disable() end end,6)
	Components.Slider(s2,"WalkSpeed",16,200,Settings.WalkSpeed,function(v)
		Settings.WalkSpeed=v; local hum=Humanoid(); if hum then hum.WalkSpeed=v end; saveSettings()
	end,7)
	Components.Slider(s2,"Sprint Speed",20,300,Settings.SprintSpeed,function(v) Settings.SprintSpeed=v; saveSettings() end,8)
	Components.Slider(s2,"JumpPower",0,400,Settings.JumpPower,function(v)
		Settings.JumpPower=v; local hum=Humanoid(); if hum then hum.UseJumpPower=true; hum.JumpPower=v end; saveSettings()
	end,9)
	Components.Button(s2,"Dash Forward",function() Dash.Do() end,10)
	Components.Slider(s2,"Dash Distance",10,100,Settings.DashDistance,function(v) Settings.DashDistance=v; saveSettings() end,11)

	local s3 = Components.Section(page, "🌐 Physics", 3)
	Components.Slider(s3,"Gravity",0,500,Settings.GravityValue,function(v) applyGravity(v) end,1,"Default: 196.2")
	Components.Slider(s3,"Character Scale",0.3,3,Settings.CharScale or 1,function(v) applyCharScale(v) end,2)
	Components.Toggle(s3,"Freeze Character",false,function(v) if v then Freeze.Enable() else Freeze.Disable() end end,3)
	Components.Button(s3,"Respawn",doRespawn,4)

	local s4 = Components.Section(page, "📌 Saved Locations", 4)
	local locNameBox = Util.New("TextBox", {
		Size=UDim2.new(1,0,0,28), BackgroundColor3=Theme.Current.Elevated,
		PlaceholderText="Location name...", Text="", Font=Enum.Font.Gotham,
		TextSize=13, TextColor3=Theme.Current.Text, PlaceholderColor3=Theme.Current.SubText,
		ClearTextOnFocus=false, LayoutOrder=1, Parent=s4,
	})
	Util.Corner(6,locNameBox); Util.Pad(locNameBox,8); themed(locNameBox,"BackgroundColor3","Elevated")
	local locListHolder = Util.New("Frame", {
		BackgroundTransparency=1, Size=UDim2.new(1,0,0,0),
		AutomaticSize=Enum.AutomaticSize.Y, LayoutOrder=4, Parent=s4,
	})
	Util.New("UIListLayout",{Parent=locListHolder,Padding=UDim.new(0,4),SortOrder=Enum.SortOrder.LayoutOrder})
	local function rebuildLocList()
		for _, c in ipairs(locListHolder:GetChildren()) do
			if c:IsA("GuiObject") then c:Destroy() end
		end
		local i = 0
		for name, loc in pairs(Settings.SavedLocations) do
			i += 1
			local row = Util.New("Frame", {
				BackgroundColor3=Theme.Current.Elevated, Size=UDim2.new(1,0,0,32),
				LayoutOrder=i, Parent=locListHolder,
			})
			Util.Corner(6,row); themed(row,"BackgroundColor3","Elevated"); Util.Pad(row,8,4,0,0)
			Util.New("TextLabel",{
				BackgroundTransparency=1, Size=UDim2.new(1,-152,1,0), Font=Enum.Font.Gotham,
				Text=name, TextSize=12, TextColor3=Theme.Current.Text,
				TextXAlignment=Enum.TextXAlignment.Left, TextTruncate=Enum.TextTruncate.AtEnd, Parent=row,
			})
			local tpBtn = Components.Button(row,"Go",function() teleportToSaved(name) end)
			tpBtn.Size=UDim2.new(0,40,0,24); tpBtn.Position=UDim2.new(1,-148,0.5,-12)
			local wpOn  = loc[4] ~= false
			local wpBtn = Util.New("TextButton",{
				Size=UDim2.new(0,68,0,24), Position=UDim2.new(1,-104,0.5,-12),
				BackgroundColor3= wpOn and Theme.Current.Accent or Theme.Current.Panel,
				Text=wpOn and "📍 On" or "📍 Off", Font=Enum.Font.GothamBold, TextSize=11,
				TextColor3=Color3.new(1,1,1), AutoButtonColor=false, Parent=row,
			})
			Util.Corner(6,wpBtn)
			wpBtn.MouseButton1Click:Connect(function()
				local l = Settings.SavedLocations[name]; if not l then return end
				l[4] = not(l[4] ~= false); saveSettings()
				toggleWaypointVisual(name, Vector3.new(l[1],l[2],l[3]), l[4])
				wpBtn.BackgroundColor3 = l[4] and Theme.Current.Accent or Theme.Current.Panel
				wpBtn.Text             = l[4] and "📍 On" or "📍 Off"
			end)
			local delBtn = Components.DangerButton(row,"✕",function() deleteSavedLocation(name); rebuildLocList() end)
			delBtn.Size=UDim2.new(0,30,0,24); delBtn.Position=UDim2.new(1,-32,0.5,-12)
		end
		if i == 0 then
			Util.New("TextLabel",{
				BackgroundTransparency=1, Size=UDim2.new(1,0,0,20), Font=Enum.Font.Gotham,
				Text="No saved locations yet.", TextSize=11, TextColor3=Theme.Current.SubText, Parent=locListHolder,
			})
		end
	end
	Components.Button(s4,"Save Current Position",function()
		local n = locNameBox.Text ~= "" and locNameBox.Text or ("Loc"..tostring(
			(function() local c=0 for _ in pairs(Settings.SavedLocations) do c+=1 end return c+1 end)()))
		saveTeleportLocation(n); locNameBox.Text=""; rebuildLocList()
	end,2)
	Components.Button(s4,"Undo Last Teleport",undoTeleport,3)
	rebuildLocList()

	local s5 = Components.Section(page, "🎥 Camera", 5)
	Components.Toggle(s5,"Freecam (WASD)",false,function(v) if v then Freecam.Enable() else Freecam.Disable() end end,1)
	Components.Toggle(s5,"Cinematic Camera",false,function(v) if v then CinematicCam.Start() else CinematicCam.Stop() end end,2)
	Components.Toggle(s5,"Orbit Camera",false,function(v) if v then OrbitCam.Enable() else OrbitCam.Disable() end end,3)
	Components.Toggle(s5,"First-Person Lock",false,function(v) if v then FPLock.Enable() else FPLock.Disable() end end,4)
	Components.Toggle(s5,"Screenshot Mode (hide UI)",false,function(v) if v then ScreenshotMode.Enable() else ScreenshotMode.Disable() end end,5)
	Components.Slider(s5,"Field of View",30,120,Settings.FOV,applyFOV,6)
	Components.Slider(s5,"Third-Person Distance",3,50,Settings.TPDistance,function(v)
		Settings.TPDistance=v; LocalPlayer.CameraMaxZoomDistance=v; saveSettings()
	end,7)
	Components.Button(s5,"Camera Shake (Preview)",function() startCameraShake(Settings.CamShakeAmt,1.5) end,8)
	Components.Slider(s5,"Shake Intensity",0.1,3,Settings.CamShakeAmt,function(v) Settings.CamShakeAmt=v; saveSettings() end,9)
	Components.Toggle(s5,"Depth of Field",false,function(v) applyDOF(v,0,0.6,50) end,10)
	Components.Slider(s5,"DOF Focus Distance",5,200,50,function(v) applyDOF(dofEffect.Enabled,0,0.6,v) end,11)
	Components.Toggle(s5,"Slow-Motion Effect",false,function(v) if v then SlowMo.Enable() else SlowMo.Disable() end end,12)
end

-- =========================================================
-- VISUALS PAGE
-- =========================================================
do
	local page = Pages.Visuals.page

	-- ESP Controls — all toggles force clearAllESP + refreshESP so every
	-- mode change (Box, Corner, Tracer, HealthBars, TeamColor, etc.) rebuilds
	-- the drawing objects correctly.
	local s1 = Components.Section(page, "👁 ESP — Target Categories", 1)
	local function espToggle(label, key, order)
		Components.Toggle(s1, label, false, function(v)
			State.ESP[key] = v
			espToggleRebuild()
		end, order)
	end
	espToggle("Players",           "Players",    1)
	espToggle("NPCs",              "NPCs",       2)
	espToggle("Items (ESP_Item)",  "Items",      3)
	espToggle("Objectives",        "Objectives", 4)
	espToggle("Vehicles",          "Vehicles",   5)

	-- Blissful drawing-style toggles
	local s1b = Components.Section(page, "👁 ESP — Drawing Styles (Blissful)", 2)
	local function espStyleToggle(label, key, order, tip)
		Components.Toggle(s1b, label, false, function(v)
			State.ESP[key] = v
			espToggleRebuild()
		end, order, tip)
	end
	espStyleToggle("Box (Quad, black border)",   "Boxes",      1, "Blissful 2D Box ESP — accurate quad with black border")
	espStyleToggle("Corner Box (rainbow + auto-thickness)", "CornerBox", 2, "Blissful Corner 2D Box ESP — 8-line L-brackets with rainbow cycling")
	espStyleToggle("Tracers (screen-bottom line)","Tracers",   3, "Blissful-style line from bottom of screen to target feet (+ black shadow)")
	espStyleToggle("Health Bars (drawn)",         "HealthBars",4, "Blissful HP bar — black bg + red→green fill on left box edge")
	espStyleToggle("Name Labels",                 "Names",     5)
	espStyleToggle("Distance Labels",             "Distances", 6)
	espStyleToggle("Health % Text",               "Health",    7)
	espStyleToggle("Skeleton Mode",               "Skeleton",  8, "Full limb skeleton R15/R6 — replaces Box when active")
	espStyleToggle("Team Colors",                 "TeamColor", 9)

	Components.Label(s1b, "ℹ Item/Objective ESP requires CollectionService tags\n'ESP_Item' / 'ESP_Objective' on your objects in Studio.", 10, true)

	-- Character Visuals
	local s2 = Components.Section(page, "✨ Character", 3)
	Components.Toggle(s2,"God Mode (visual only)",false,function(v) if v then GodModeVisual.Enable() else GodModeVisual.Disable() end end,1,"Locks local health to max — cosmetic only")
	Components.Toggle(s2,"Invisible Character",false,function(v) if v then Invisible.Enable() else Invisible.Disable() end end,2)
	Components.Toggle(s2,"Rainbow Character",false,function(v) if v then RainbowChar.Enable() else RainbowChar.Disable() end end,3)
	Components.Toggle(s2,"Trail",false,function(v) if v then TrailCreator.Enable() else TrailCreator.Disable() end end,4)
	Components.Toggle(s2,"Aura Particles",false,function(v) if v then AuraEffect.Enable() else AuraEffect.Disable() end end,5)
	local ftBox = Util.New("TextBox", {
		Size=UDim2.new(1,0,0,28), BackgroundColor3=Theme.Current.Elevated,
		PlaceholderText="Floating text...", Text="", Font=Enum.Font.Gotham, TextSize=13,
		TextColor3=Theme.Current.Text, PlaceholderColor3=Theme.Current.SubText,
		ClearTextOnFocus=false, LayoutOrder=6, Parent=s2,
	})
	Util.Corner(6,ftBox); Util.Pad(ftBox,8); themed(ftBox,"BackgroundColor3","Elevated")
	Components.Button(s2,"Show Floating Text",function()
		createFloatingText(ftBox.Text ~= "" and ftBox.Text or "Hello!", Theme.Current.Accent)
	end,7)
	Components.Button(s2,"Spawn Particles on Self",function() spawnSelfParticles(Theme.Current.Accent) end,8)

	-- World Visuals
	local s3 = Components.Section(page, "🌍 World Visuals", 4)
	Components.Toggle(s3,"Fullbright",false,function(v) if v then Fullbright.Enable() else Fullbright.Disable() end end,1)
	Components.Toggle(s3,"X-Ray Mode",false,function(v) if v then XRay.Enable() else XRay.Disable() end end,2)
	Components.Toggle(s3,"Highlight Interactables",false,function(v) if v then HighlightInteractables.Enable() else HighlightInteractables.Disable() end end,3)
	Components.Toggle(s3,"Remove Visual Clutter",false,function(v) if v then RemoveClutter.Enable() else RemoveClutter.Disable() end end,4)
	Components.Toggle(s3,"Show HUD Overlay",Settings.OverlayOn,function(v) Settings.OverlayOn=v; saveSettings() end,5)
end

-- =========================================================
-- WORLD PAGE
-- =========================================================
do
	local page = Pages.World.page
	local s1 = Components.Section(page,"🌤 Time & Lighting",1)
	Components.Slider(s1,"Time of Day (0-24)",0,24,Lighting.ClockTime,function(v) setTime(v) end,1,"0=midnight, 12=noon")
	Components.Slider(s1,"Brightness",0,5,Lighting.Brightness,function(v) Lighting.Brightness=v end,2)
	Components.Toggle(s1,"Sun Rays Effect",false,function(v) SunRaysEffect.Enabled=v end,3)
	local s2 = Components.Section(page,"🌦 Weather",2)
	for i, w in ipairs({"Clear","Cloudy","Rainy","Storm","Foggy"}) do
		Components.Button(s2,w,function() setWeather(w) end,i)
	end
	local s3 = Components.Section(page,"🌫 Fog",3)
	Components.Slider(s3,"Fog Start",0,5000,Lighting.FogStart,function(v) Lighting.FogStart=v end,1)
	Components.Slider(s3,"Fog End",0,5000,math.min(Lighting.FogEnd,5000),function(v) Lighting.FogEnd=v end,2)
	local s4 = Components.Section(page,"🎨 Color Correction",4)
	Components.Toggle(s4,"Enable Color Correction",false,function(v) ColorCorrection.Enabled=v end,1)
	Components.Slider(s4,"Brightness",-1,1,0,function(v) ColorCorrection.Brightness=v end,2)
	Components.Slider(s4,"Contrast",  -1,1,0,function(v) ColorCorrection.Contrast=v end,3)
	Components.Slider(s4,"Saturation",-1,1,0,function(v) ColorCorrection.Saturation=v end,4)
	local s5 = Components.Section(page,"⚡ Effects",5)
	Components.Button(s5,"⚡ Lightning Strike",doLightningStrike,1)
	Components.Button(s5,"☄ Meteor Drop",doMeteorEffect,2)
	Components.Button(s5,"🌊 Earthquake (3s)",function() doEarthquake(3) end,3)
	Components.Button(s5,"💥 Explosion at Me",function()
		local root=HRP(); if not root then return end
		local e=Util.New("Explosion",{
			Position=root.Position, BlastPressure=0, BlastRadius=10,
			ExplosionType=Enum.ExplosionType.NoCraters, Parent=Workspace,
		})
		task.delay(2,function() if e and e.Parent then e:Destroy() end end)
	end,4)
	Components.Button(s5,"🌈 Rainbow Particles",function()
		local h=0
		for i=1,8 do task.delay(i*0.15,function() h=(h+45)%360; spawnSelfParticles(Color3.fromHSV(h/360,1,1)) end) end
	end,5)
end

-- =========================================================
-- PLAYERS PAGE
-- =========================================================
local PlayerListRefresh
do
	local page    = Pages.Players.page
	local searchSec = Components.Section(page,nil,1)
	local searchBox = Util.New("TextBox",{
		Size=UDim2.new(1,0,0,30), BackgroundColor3=Theme.Current.Elevated,
		PlaceholderText="Search players...", Text="", Font=Enum.Font.Gotham,
		TextSize=13, TextColor3=Theme.Current.Text, PlaceholderColor3=Theme.Current.SubText,
		ClearTextOnFocus=false, Parent=searchSec,
	})
	Util.Corner(8,searchBox); Util.Pad(searchBox,10); themed(searchBox,"BackgroundColor3","Elevated")
	local listSec    = Components.Section(page,"Players in Server",2)
	local listHolder = Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,0),AutomaticSize=Enum.AutomaticSize.Y,Parent=listSec})
	Util.New("UIListLayout",{Parent=listHolder,Padding=UDim.new(0,6),SortOrder=Enum.SortOrder.LayoutOrder})
	local rows = {}
	local function buildRow(plr)
		local row = Util.New("Frame",{BackgroundColor3=Theme.Current.Elevated,Size=UDim2.new(1,0,0,44),Parent=listHolder})
		Util.Corner(8,row); themed(row,"BackgroundColor3","Elevated"); Util.Pad(row,10)
		Util.New("ImageLabel",{BackgroundColor3=Theme.Current.Panel,Size=UDim2.new(0,28,0,28),Position=UDim2.new(0,0,0.5,-14),
			Image="rbxthumb://type=AvatarHeadShot&id="..tostring(plr.UserId).."&w=48&h=48",Parent=row})
		Util.New("TextLabel",{BackgroundTransparency=1,Position=UDim2.new(0,36,0,4),Size=UDim2.new(0.4,0,0,16),
			Font=Enum.Font.GothamBold,Text=plr.DisplayName,TextColor3=Theme.Current.Text,TextSize=13,
			TextXAlignment=Enum.TextXAlignment.Left,Parent=row})
		Util.New("TextLabel",{BackgroundTransparency=1,Position=UDim2.new(0,36,0,20),Size=UDim2.new(0.4,0,0,14),
			Font=Enum.Font.Gotham,Text="@"..plr.Name,TextColor3=Theme.Current.SubText,TextSize=11,
			TextXAlignment=Enum.TextXAlignment.Left,Parent=row})
		local tpBtn = Components.Button(row,"TP",function() teleportTo(plr) end)
		tpBtn.Size=UDim2.new(0,36,0,26); tpBtn.Position=UDim2.new(1,-130,0.5,-13)
		local specBtn = Components.Button(row,"Spec",function()
			if State.Spectating==plr then Spectate.Stop(); specBtn.Text="Spec"
			else Spectate.Start(plr); specBtn.Text="Unspec" end
		end)
		specBtn.Size=UDim2.new(0,52,0,26); specBtn.Position=UDim2.new(1,-86,0.5,-13)
		local stopBtn = Components.Button(row,"📷✕",function() if State.Spectating then Spectate.Stop(); specBtn.Text="Spec" end end)
		stopBtn.Size=UDim2.new(0,28,0,26); stopBtn.Position=UDim2.new(1,-28,0.5,-13)
		stopBtn.BackgroundColor3=Theme.Current.Bad
		return row
	end
	PlayerListRefresh = function()
		for _, r in pairs(rows) do if r and r.Parent then r:Destroy() end end
		rows={}
		local filter=searchBox.Text:lower()
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LocalPlayer and (filter=="" or plr.Name:lower():find(filter,1,true) or plr.DisplayName:lower():find(filter,1,true)) then
				rows[plr]=buildRow(plr)
			end
		end
	end
	searchBox:GetPropertyChangedSignal("Text"):Connect(PlayerListRefresh)
	Players.PlayerAdded:Connect(function() task.wait(0.5); PlayerListRefresh() end)
	Players.PlayerRemoving:Connect(function() task.wait(); PlayerListRefresh() end)
	PlayerListRefresh()
	local npcSec = Components.Section(page,"🤖 NPC Tracker",3)
	local npcLog = Components.LogBox(npcSec,1)
	Components.Button(npcSec,"Refresh NPC List",function()
		npcLog.Clear()
		local found=0
		for _, desc in ipairs(Workspace:GetDescendants()) do
			if desc:IsA("Humanoid") and not Players:GetPlayerFromCharacter(desc.Parent) then
				local root=desc.Parent and desc.Parent:FindFirstChild("HumanoidRootPart")
				local myR=HRP()
				local dist=root and myR and math.floor((root.Position-myR.Position).Magnitude) or -1
				npcLog.Append((desc.Parent and desc.Parent.Name or "?").." | HP:"..math.floor(desc.Health).."  | "..dist.."m", Theme.Current.SubText)
				found+=1
			end
		end
		if found==0 then npcLog.Append("No NPCs found.", Theme.Current.SubText) end
	end,2)
end

-- =========================================================
-- DEVELOPER PAGE
-- =========================================================
do
	local page = Pages.Developer.page
	local s1   = Components.Section(page,"📡 Remote Loggers",1)
	local remLog = Components.LogBox(s1,1)
	Components.Toggle(s1,"Log RemoteEvents",false,function(v) if v then RemoteLogger.Enable(remLog) else RemoteLogger.Disable() end end,2)
	Components.Button(s1,"Clear Log",function() remLog.Clear() end,3)

	local s2 = Components.Section(page,"📊 Runtime Stats",2)
	local statsLog = Components.LogBox(s2,1)
	Components.Button(s2,"Refresh Stats",function()
		statsLog.Clear()
		statsLog.Append("Parts in Workspace: "..#Workspace:GetDescendants(), Theme.Current.Text)
		statsLog.Append("Lua Memory: "..Util.Round(collectgarbage("count")/1024,2).." MB", Theme.Current.Text)
		statsLog.Append("Players: "..#Players:GetPlayers(), Theme.Current.Text)
		statsLog.Append("PlaceId: "..tostring(game.PlaceId), Theme.Current.Text)
		local ok, ping = pcall(function() return Stats.Network.ServerStatsItem["Data Ping"]:GetValue() end)
		statsLog.Append("Ping: "..(ok and math.floor(ping).."ms" or "n/a"), Theme.Current.Text)
		local partCount=0
		for _, d in ipairs(Workspace:GetDescendants()) do if d:IsA("BasePart") then partCount+=1 end end
		statsLog.Append("BaseParts: "..partCount, Theme.Current.Text)
	end,2)

	local s3 = Components.Section(page,"🔍 Instance Explorer",3)
	local explorerLog = Components.LogBox(s3,1); explorerLog.frame.Size=UDim2.new(1,0,0,180)
	local explorerPath = Util.New("TextBox",{
		Size=UDim2.new(1,0,0,28), BackgroundColor3=Theme.Current.Elevated,
		PlaceholderText="e.g. Workspace.Map.Baseplate", Text="", Font=Enum.Font.Code, TextSize=12,
		TextColor3=Theme.Current.Text, PlaceholderColor3=Theme.Current.SubText,
		ClearTextOnFocus=false, LayoutOrder=2, Parent=s3,
	})
	Util.Corner(6,explorerPath); Util.Pad(explorerPath,8); themed(explorerPath,"BackgroundColor3","Elevated")
	Components.Button(s3,"Explore Path",function()
		explorerLog.Clear()
		local ok,result=pcall(function()
			local parts={}
			for seg in explorerPath.Text:gmatch("[^%.]+") do table.insert(parts,seg) end
			local cur=game
			for _, seg in ipairs(parts) do cur=cur:FindFirstChild(seg) or cur[seg] end
			return cur
		end)
		if ok and result then
			explorerLog.Append("["..result.ClassName.."] "..result:GetFullName(), Theme.Current.Accent)
			for _, child in ipairs(result:GetChildren()) do
				explorerLog.Append("  └ ["..child.ClassName.."] "..child.Name, Theme.Current.SubText)
			end
		else explorerLog.Append("Not found: "..explorerPath.Text, Theme.Current.Bad) end
	end,3)

	local s4 = Components.Section(page,"🖱 Object Inspector",4)
	local inspLog = Components.LogBox(s4,1)
	Components.Toggle(s4,"Click to Inspect Parts",false,function(v) if v then startInspector(inspLog) else stopInspector() end end,2)

	local s5 = Components.Section(page,"🚨 Error Log",5)
	local errLog = Components.LogBox(s5,1)
	Components.Button(s5,"Refresh Errors",function()
		errLog.Clear()
		if #_errorLog==0 then errLog.Append("No errors captured.", Theme.Current.Good)
		else for _, e in ipairs(_errorLog) do errLog.Append(e, Theme.Current.Bad) end end
	end,2)

	local s6 = Components.Section(page,"🧪 Testing Tools",6)
	Components.Toggle(s6,"Simulate Lag (~200ms)",false,function(v) if v then LagSim.Enable(200) else LagSim.Disable() end end,1)
	Components.Toggle(s6,"Collision Visualizer",false,function(v) if v then CollisionViz.Enable() else CollisionViz.Disable() end end,2)
	Components.Toggle(s6,"Hitbox Visualizer",false,function(v) if v then HitboxViz.Enable() else HitboxViz.Disable() end end,3)
	Components.Button(s6,"Count All Parts",function()
		local n=0
		for _, d in ipairs(Workspace:GetDescendants()) do if d:IsA("BasePart") then n+=1 end end
		Notify.Push("Part Count","Workspace contains "..n.." BaseParts","Info",4)
	end,4)

	local s7 = Components.Section(page,"🗺 Pathfinding Visualizer",7)
	Components.Button(s7,"Visualize Path to Mouse Target",function()
		local root=HRP(); if not root then return end
		local PathfindingService=game:GetService("PathfindingService")
		local path=PathfindingService:CreatePath()
		local ok=pcall(function() path:ComputeAsync(root.Position, Mouse.Hit.Position) end)
		if not ok or path.Status ~= Enum.PathStatus.Success then
			Notify.Push("Pathfinding","Path could not be computed","Error"); return
		end
		for _, wp in ipairs(path:GetWaypoints()) do
			local dot=Util.New("Part",{
				Anchored=true,CanCollide=false,Size=Vector3.new(0.6,0.6,0.6),
				Shape=Enum.PartType.Ball,Material=Enum.Material.Neon,
				Color=Theme.Current.Accent,CFrame=CFrame.new(wp.Position),Parent=Workspace,
			})
			task.delay(5,function() if dot and dot.Parent then dot:Destroy() end end)
		end
		Notify.Push("Pathfinding",#path:GetWaypoints().." waypoints (5s)","Info",3)
	end,1)
end

-- =========================================================
-- QoL PAGE
-- =========================================================
do
	local page = Pages.QoL.page
	local s1 = Components.Section(page,"⭐ Favorites",1)
	local favHolder = Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,0),AutomaticSize=Enum.AutomaticSize.Y,LayoutOrder=1,Parent=s1})
	Util.New("UIListLayout",{Parent=favHolder,Padding=UDim.new(0,4)})
	local function rebuildFavs()
		for _, c in ipairs(favHolder:GetChildren()) do if c:IsA("GuiObject") then c:Destroy() end end
		local any=false
		for name, v in pairs(Favorites) do
			if v then
				any=true
				local row=Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,24),Parent=favHolder})
				Util.New("TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,-60,1,0),Font=Enum.Font.Gotham,
					Text="★ "..name,TextSize=12,TextColor3=Theme.Current.Accent,TextXAlignment=Enum.TextXAlignment.Left,Parent=row})
				local tab="Movement"
				for _, feat in ipairs(AllFeatureLabels) do if feat.name==name then tab=feat.tab; break end end
				local goBtn=Components.Button(row,"Go →",function() SwitchPage(tab) end)
				goBtn.Size=UDim2.new(0,50,0,22); goBtn.Position=UDim2.new(1,-50,0.5,-11)
			end
		end
		if not any then
			Util.New("TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,20),Font=Enum.Font.Gotham,
				Text="No favorites yet. Click ☆ next to any toggle.",TextSize=11,TextColor3=Theme.Current.SubText,Parent=favHolder})
		end
	end
	rebuildFavs()
	Components.Button(s1,"Refresh Favorites",rebuildFavs,2)

	local s2=Components.Section(page,"🕐 Recent Pages",2)
	local recentHolder=Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,0),AutomaticSize=Enum.AutomaticSize.Y,LayoutOrder=1,Parent=s2})
	Util.New("UIListLayout",{Parent=recentHolder,Padding=UDim.new(0,4)})
	local function rebuildRecents()
		for _, c in ipairs(recentHolder:GetChildren()) do if c:IsA("GuiObject") then c:Destroy() end end
		local seen={} local i=0
		for _, name in ipairs(_pageHistory) do
			if not seen[name] then seen[name]=true; i+=1; if i>5 then break end
				local row=Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,24),Parent=recentHolder})
				Util.New("TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,-60,1,0),Font=Enum.Font.Gotham,
					Text=i..". "..name,TextSize=12,TextColor3=Theme.Current.SubText,TextXAlignment=Enum.TextXAlignment.Left,Parent=row})
				local goBtn=Components.Button(row,"Go",function() SwitchPage(name) end)
				goBtn.Size=UDim2.new(0,40,0,22); goBtn.Position=UDim2.new(1,-40,0.5,-11)
			end
		end
	end
	Components.Button(s2,"Refresh Recents",rebuildRecents,2)

	local s3=Components.Section(page,"🔔 Notification History",3)
	local nhLog=Components.LogBox(s3,1); nhLog.frame.Size=UDim2.new(1,0,0,160)
	Components.Button(s3,"Load Notification History",function()
		nhLog.Clear()
		if #Settings.NotifyHistory==0 then nhLog.Append("No notifications yet.",Theme.Current.SubText); return end
		local kinds={Info=Theme.Current.Accent,Success=Theme.Current.Good,Error=Theme.Current.Bad,Warn=Theme.Current.Accent}
		for _, e in ipairs(Settings.NotifyHistory) do
			nhLog.Append("["..e.kind.."] "..e.title..(e.message~="" and ": "..e.message or ""), kinds[e.kind] or Theme.Current.Text)
		end
	end,2)
	Components.Button(s3,"Clear History",function() Settings.NotifyHistory={}; saveSettings(); nhLog.Clear() end,3)

	local s4=Components.Section(page,"💾 Profiles",4)
	local profileNameBox=Util.New("TextBox",{
		Size=UDim2.new(1,0,0,28),BackgroundColor3=Theme.Current.Elevated,PlaceholderText="Profile name...",
		Text="",Font=Enum.Font.Gotham,TextSize=13,TextColor3=Theme.Current.Text,
		PlaceholderColor3=Theme.Current.SubText,ClearTextOnFocus=false,LayoutOrder=1,Parent=s4,
	})
	Util.Corner(6,profileNameBox); Util.Pad(profileNameBox,8); themed(profileNameBox,"BackgroundColor3","Elevated")
	Components.Button(s4,"Save Current Settings as Profile",function()
		local n=profileNameBox.Text
		if n=="" then Notify.Push("Profile","Enter a name first","Warn"); return end
		Settings.Profiles[n]=Util.DeepCopy(Settings); Settings.Profiles[n].Profiles=nil
		saveSettings(); Notify.Push("Profile Saved",n,"Success",2)
	end,2)
	local profileListHolder=Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,0),AutomaticSize=Enum.AutomaticSize.Y,LayoutOrder=3,Parent=s4})
	Util.New("UIListLayout",{Parent=profileListHolder,Padding=UDim.new(0,4)})
	local function rebuildProfiles()
		for _, c in ipairs(profileListHolder:GetChildren()) do if c:IsA("GuiObject") then c:Destroy() end end
		local hasAny=false
		for name, _ in pairs(Settings.Profiles or {}) do
			hasAny=true
			local row=Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,28),Parent=profileListHolder})
			Util.New("TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,-100,1,0),Font=Enum.Font.Gotham,
				Text=name,TextSize=12,TextColor3=Theme.Current.SubText,TextXAlignment=Enum.TextXAlignment.Left,Parent=row})
			local loadBtn=Components.Button(row,"Load",function()
				local p=Settings.Profiles[name]
				if p then
					for k,v in pairs(p) do if k~="Profiles" and k~="NotifyHistory" then Settings[k]=v end end
					saveSettings(); applyMovementStats(); applyFOV(Settings.FOV); Theme.Set(Settings.Theme)
					Notify.Push("Profile Loaded",name,"Success",2)
				end
			end)
			loadBtn.Size=UDim2.new(0,50,0,24); loadBtn.Position=UDim2.new(1,-100,0.5,-12)
			local delBtn=Components.DangerButton(row,"✕",function() Settings.Profiles[name]=nil; saveSettings(); rebuildProfiles() end)
			delBtn.Size=UDim2.new(0,30,0,24); delBtn.Position=UDim2.new(1,-44,0.5,-12)
		end
		if not hasAny then
			Util.New("TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,16),Font=Enum.Font.Gotham,
				Text="No profiles saved yet.",TextSize=11,TextColor3=Theme.Current.SubText,Parent=profileListHolder})
		end
	end
	Components.Button(s4,"Refresh Profile List",rebuildProfiles,4); rebuildProfiles()

	local s5=Components.Section(page,"⚡ Quick Actions",5)
	Components.Button(s5,"Reset Gravity to Default",function() applyGravity(196.2) end,1)
	Components.Button(s5,"Reset FOV to 70",function() applyFOV(70) end,2)
	Components.Button(s5,"Reset Character Scale",function() applyCharScale(1) end,3)
	Components.Button(s5,"Set Time to Noon",function() setTime(12) end,4)
	Components.Button(s5,"Set Time to Midnight",function() setTime(0) end,5)
	Components.Button(s5,"Clear All ESP",function() clearAllESP() end,6)
	Components.Button(s5,"Stop All Active Features",function()
		Fly.Disable(); Noclip.Disable(); InfJump.Disable(); DoubleJump.Disable()
		WallClimb.Disable(); SwimInAir.Disable(); Freeze.Disable()
		Freecam.Disable(); Fullbright.Disable(); Invisible.Disable()
		GodModeVisual.Disable(); RainbowChar.Disable(); TrailCreator.Disable()
		AuraEffect.Disable(); XRay.Disable(); HighlightInteractables.Disable()
		RemoveClutter.Disable(); clearAllESP()
		Notify.Push("Quick Actions","All features stopped","Success",2)
	end,7)
end

-- =========================================================
-- KEYBINDS PAGE
-- =========================================================
do
	local page = Pages.Keybinds.page
	local sec  = Components.Section(page,"Feature Keybinds",1)
	local function kbRow(label, featureId, defaultKey)
		local r=Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,28),Parent=sec})
		Util.New("TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,-100,1,0),Font=Enum.Font.Gotham,
			Text=label,TextSize=13,TextColor3=Theme.Current.SubText,TextXAlignment=Enum.TextXAlignment.Left,Parent=r})
		local kb=Components.KeybindButton(r,featureId,defaultKey)
		kb.Position=UDim2.new(1,-90,0.5,-13)
		return kb
	end
	kbRow("Fly",            "Fly",          "F")
	kbRow("Noclip",         "Noclip",       "N")
	kbRow("Sprint",         "Sprint",       "LeftShift")
	kbRow("Dash",           "Dash",         "Q")
	kbRow("Freecam",        "Freecam",      "C")
	kbRow("Fullbright",     "Fullbright",   "B")
	kbRow("Invisible",      "Invisible",    "H")
	kbRow("God Mode",       "GodMode",      "G")
	kbRow("Freeze",         "Freeze",       "Z")
	kbRow("Screenshot Mode","Screenshot",   "F10")
	kbRow("Cinematic Cam",  "CinematicCam", "K")
	kbRow("Orbit Cam",      "OrbitCam",     "O")
	kbRow("Respawn",        "Respawn",      "R")
	kbRow("Undo Teleport",  "UndoTeleport", "U")
	local sec2=Components.Section(page,"Panel Controls",2)
	local r2=Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,28),Parent=sec2})
	Util.New("TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,-100,1,0),Font=Enum.Font.Gotham,
		Text="Show / Hide Panel",TextSize=13,TextColor3=Theme.Current.SubText,TextXAlignment=Enum.TextXAlignment.Left,Parent=r2})
	local toggleKeyBtn=Util.New("TextButton",{
		Size=UDim2.new(0,90,0,26),Position=UDim2.new(1,-90,0.5,-13),
		BackgroundColor3=Theme.Current.Elevated,Text=Settings.ToggleKey,
		Font=Enum.Font.GothamBold,TextSize=12,TextColor3=Theme.Current.Text,
		AutoButtonColor=false,Parent=r2,
	})
	Util.Corner(6,toggleKeyBtn); themed(toggleKeyBtn,"BackgroundColor3","Elevated")
	toggleKeyBtn.MouseButton1Click:Connect(function()
		toggleKeyBtn.Text="..."
		local conn
		conn=UserInputService.InputBegan:Connect(function(input)
			if input.UserInputType==Enum.UserInputType.Keyboard then
				conn:Disconnect()
				Settings.ToggleKey=input.KeyCode.Name
				toggleKeyBtn.Text=input.KeyCode.Name
				saveSettings()
			end
		end)
	end)
end

-- =========================================================
-- SETTINGS PAGE
-- =========================================================
do
	local page=Pages.Settings.page
	local s1=Components.Section(page,"🎨 Theme",1)
	local swatchRow=Util.New("Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,40),LayoutOrder=1,Parent=s1})
	Util.New("UIListLayout",{Parent=swatchRow,FillDirection=Enum.FillDirection.Horizontal,
		Padding=UDim.new(0,6),VerticalAlignment=Enum.VerticalAlignment.Center})
	for name, preset in pairs(Theme.Presets) do
		local swatch=Util.New("TextButton",{Size=UDim2.new(0,72,0,34),BackgroundColor3=preset.Accent,
			Text=name,Font=Enum.Font.GothamBold,TextSize=10,TextColor3=Color3.new(1,1,1),AutoButtonColor=false,Parent=swatchRow})
		Util.Corner(8,swatch)
		swatch.MouseButton1Click:Connect(function() Theme.Set(name) end)
	end
	local s1b=Components.Section(page,"🖌 Theme Editor",2)
	for _, key in ipairs({"Accent","Background","Panel","Elevated","Text","SubText","Good","Bad"}) do
		Components.ColorPicker(s1b, key, Theme.Current[key], function(c)
			Theme.Current[key]=c
			for _, fn in ipairs(Theme._listeners) do pcall(fn, Theme.Current) end
		end)
	end
	local s2=Components.Section(page,"⚙ Preferences",3)
	Components.Toggle(s2,"Enable Notifications",Settings.NotificationsOn,function(v) Settings.NotificationsOn=v; saveSettings() end,1)
	Components.Toggle(s2,"Blur Background on Open",false,function(v) setBlur(v) end,2)
	local s3=Components.Section(page,"💾 Data",4)
	Components.Button(s3,"Reset All Settings",function()
		for k,v in pairs(DEFAULT_SETTINGS) do Settings[k]=Util.DeepCopy(v) end
		saveSettings(); Notify.Push("Settings Reset","Defaults restored","Info")
	end,1)
	Components.Button(s3,"Export Settings to Chat",function()
		local ok,json=pcall(function() return HttpService:JSONEncode(Settings) end)
		if ok then StarterGui:SetCore("ChatMakeSystemMessage",{Text=json:sub(1,200),Color=Color3.fromRGB(200,200,255)}) end
	end,2)
	Util.New("TextLabel",{
		BackgroundTransparency=1,Size=UDim2.new(1,0,0,16),LayoutOrder=3,
		Font=Enum.Font.Gotham,TextSize=11,TextColor3=Theme.Current.SubText,
		TextXAlignment=Enum.TextXAlignment.Left,
		Text=canPersist() and "✅ Persistence: enabled (writefile)" or "⚠ Persistence: session-only",
		Parent=s3,
	})
end

SwitchPage("Movement")

----------------------------------------------------------------------------
-- HUD OVERLAY
----------------------------------------------------------------------------
local Overlay = Util.New("Frame", {
	Name                  = "Overlay",
	Position              = UDim2.new(0,16,0,16),
	Size                  = UDim2.new(0,230,0,140),
	BackgroundColor3      = Theme.Current.Panel,
	BackgroundTransparency= 0.12,
	Visible               = Settings.OverlayOn,
	Parent                = ScreenGui,
})
Util.Corner(10, Overlay)
themed(Overlay, "BackgroundColor3", "Panel")
Util.Stroke(Overlay, Theme.Current.Accent, 1, 0.7)
Util.Pad(Overlay, 12, 12, 10, 10)
Util.New("UIListLayout", { Parent=Overlay, Padding=UDim.new(0,3) })

local function overlayLine(name)
	return Util.New("TextLabel", {
		BackgroundTransparency= 1,
		Size                  = UDim2.new(1,0,0,16),
		Font                  = Enum.Font.Code,
		TextSize              = 12,
		TextColor3            = Theme.Current.Text,
		TextXAlignment        = Enum.TextXAlignment.Left,
		Text                  = name..": --",
		Parent                = Overlay,
	})
end
local fpsLbl   = overlayLine("FPS")
local pingLbl  = overlayLine("Ping")
local coordLbl = overlayLine("Pos")
local gravLbl  = overlayLine("Gravity")
local wsLbl    = overlayLine("Speed")
local placeLbl = overlayLine("Place")
local jobLbl   = overlayLine("Server")
placeLbl.Text = "Place: "..tostring(game.PlaceId)
jobLbl.Text   = "Server: "..(game.JobId ~= "" and game.JobId:sub(1,8) or "Studio")

Theme.OnChange(function(c)
	for _, lbl in ipairs(Overlay:GetChildren()) do
		if lbl:IsA("TextLabel") then lbl.TextColor3=c.Text end
	end
end)

local frameCount, lastClock = 0, os.clock()
RunService.RenderStepped:Connect(function()
	frameCount += 1
	local now = os.clock()
	if now - lastClock >= 0.5 then
		fpsLbl.Text        = "FPS: "..math.floor(frameCount/(now-lastClock))
		frameCount, lastClock = 0, now
	end
	local root = HRP()
	if root then
		local p = root.Position
		coordLbl.Text = string.format("Pos: %d, %d, %d", p.X, p.Y, p.Z)
	end
	gravLbl.Text   = "Gravity: "..math.floor(Workspace.Gravity)
	wsLbl.Text     = "Speed: "..(Humanoid() and math.floor(Humanoid().WalkSpeed) or "--")
	Overlay.Visible= Settings.OverlayOn
end)

task.spawn(function()
	while task.wait(1) do
		local ok, ping = pcall(function()
			return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
		end)
		pingLbl.Text = "Ping: "..(ok and math.floor(ping).."ms" or "n/a")
	end
end)

----------------------------------------------------------------------------
-- COMMAND PALETTE
----------------------------------------------------------------------------
local CommandIndex = {}
for _, feat in ipairs(AllFeatureLabels) do
	table.insert(CommandIndex, { name=feat.name, run=function() SwitchPage(feat.tab) end })
end
for name, _ in pairs(Pages) do
	table.insert(CommandIndex, { name="Go to "..name, run=function() SwitchPage(name) end })
end

local Palette = Util.New("Frame", {
	Name             = "CommandPalette",
	Size             = UDim2.new(0,480,0,0),
	AutomaticSize    = Enum.AutomaticSize.Y,
	Position         = UDim2.new(0.5,-240,0.15,0),
	BackgroundColor3 = Theme.Current.Panel,
	Visible          = false,
	ZIndex           = 50,
	Parent           = ScreenGui,
})
Util.Corner(12, Palette)
themed(Palette, "BackgroundColor3", "Panel")
Util.Stroke(Palette, Theme.Current.Accent, 1, 0.5)
Util.Pad(Palette, 14, 14, 14, 14)
Util.New("UIListLayout", { Parent=Palette, Padding=UDim.new(0,6) })

local paletteInput = Util.New("TextBox", {
	Size              = UDim2.new(1,0,0,36),
	BackgroundColor3  = Theme.Current.Elevated,
	PlaceholderText   = "Type a command or feature name...",
	Text              = "",
	Font              = Enum.Font.Gotham,
	TextSize          = 14,
	TextColor3        = Theme.Current.Text,
	PlaceholderColor3 = Theme.Current.SubText,
	ZIndex            = 51,
	Parent            = Palette,
})
Util.Corner(8, paletteInput)
Util.Pad(paletteInput, 10)
themed(paletteInput, "BackgroundColor3", "Elevated")

local resultsHolder = Util.New("Frame", {
	BackgroundTransparency= 1,
	Size                  = UDim2.new(1,0,0,0),
	AutomaticSize         = Enum.AutomaticSize.Y,
	ZIndex                = 51,
	Parent                = Palette,
})
Util.New("UIListLayout", { Parent=resultsHolder, Padding=UDim.new(0,4) })

local function closePalette() Palette.Visible=false end

local function refreshPaletteResults()
	for _, c in ipairs(resultsHolder:GetChildren()) do if c:IsA("GuiObject") then c:Destroy() end end
	local query = paletteInput.Text:lower()
	local count = 0
	for _, cmd in ipairs(CommandIndex) do
		if query=="" or cmd.name:lower():find(query,1,true) then
			count+=1; if count>8 then break end
			local btn=Util.New("TextButton",{
				Size=UDim2.new(1,0,0,30), BackgroundColor3=Theme.Current.Elevated,
				Text="  "..cmd.name, Font=Enum.Font.GothamMedium, TextSize=13,
				TextColor3=Theme.Current.Text, TextXAlignment=Enum.TextXAlignment.Left,
				AutoButtonColor=false, ZIndex=51, Parent=resultsHolder,
			})
			Util.Corner(6,btn)
			btn.MouseEnter:Connect(function() Util.Tween(btn,{BackgroundColor3=Theme.Current.AccentSoft},0.12) end)
			btn.MouseLeave:Connect(function() Util.Tween(btn,{BackgroundColor3=Theme.Current.Elevated},0.12) end)
			btn.MouseButton1Click:Connect(function() cmd.run(); closePalette() end)
		end
	end
end
paletteInput:GetPropertyChangedSignal("Text"):Connect(refreshPaletteResults)
local function openPalette()
	Palette.Visible=true; paletteInput.Text=""; refreshPaletteResults(); paletteInput:CaptureFocus()
end

----------------------------------------------------------------------------
-- WINDOW CONTROLS
----------------------------------------------------------------------------
local fullHeight = Window.Size

MinimizeBtn.MouseButton1Click:Connect(function()
	State.Minimized = not State.Minimized
	if State.Minimized then
		Util.Tween(Window,{Size=UDim2.new(0,860,0,46)},0.25)
		task.delay(0.22,function() Body.Visible=false end)
	else
		Body.Visible=true
		Util.Tween(Window,{Size=fullHeight},0.25)
	end
end)

BlurBtn.MouseButton1Click:Connect(function() setBlur(not _blurEnabled) end)

local _docked=false
DockBtn.MouseButton1Click:Connect(function()
	_docked=not _docked
	if _docked then
		Util.Tween(Window,{Position=UDim2.new(0,0,0,0),Size=UDim2.new(0,300,1,0)},0.3)
	else
		Util.Tween(Window,{Position=UDim2.new(0.5,-430,0.5,-270),Size=fullHeight},0.3)
	end
end)

local function setPanelVisible(v)
	State.PanelVisible=v
	if v then
		Window.Visible=true
		Window.Size=UDim2.new(0,860,0,0)
		Window.BackgroundTransparency=1
		Util.Tween(Window,{Size=State.Minimized and UDim2.new(0,860,0,46) or fullHeight,BackgroundTransparency=0},0.25)
	else
		Util.Tween(Window,{Size=UDim2.new(0,860,0,0),BackgroundTransparency=1},0.2)
		task.delay(0.21,function() if not State.PanelVisible then Window.Visible=false end end)
	end
end

CloseBtn.MouseButton1Click:Connect(function() setPanelVisible(false) end)

----------------------------------------------------------------------------
-- GLOBAL KEYBIND ROUTER
----------------------------------------------------------------------------
local featureToggleMap = {
	Fly        = { flag="Flying",      feature=Fly },
	Noclip     = { flag="Noclipping",  feature=Noclip },
	Freecam    = { flag="Freecamming", feature=Freecam },
	Invisible  = { flag="Invisible",   feature=Invisible },
	GodMode    = { flag="GodMode",     feature=GodModeVisual },
	Freeze     = { flag="Frozen",      feature=Freeze },
}

UserInputService.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
	local keyName = input.KeyCode.Name
	if keyName == Settings.ToggleKey then setPanelVisible(not State.PanelVisible); return end
	if keyName == "P" and (UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.RightControl)) then
		if Palette.Visible then closePalette() else openPalette() end; return
	end
	if keyName == "Escape" and Palette.Visible then closePalette(); return end
	for featureId, bind in pairs(Settings.Keybinds) do
		if keyName == bind then
			if featureToggleMap[featureId] then
				local m=featureToggleMap[featureId]
				if State[m.flag] then m.feature.Disable() else m.feature.Enable() end
			elseif featureId=="Fullbright" then
				if State.Fullbright then Fullbright.Disable() else Fullbright.Enable() end
			elseif featureId=="Dash"          then Dash.Do()
			elseif featureId=="Sprint"        then Sprint.Toggle(not State.Sprinting)
			elseif featureId=="Screenshot"    then if State.ScreenshotMode then ScreenshotMode.Disable() else ScreenshotMode.Enable() end
			elseif featureId=="CinematicCam"  then if CinematicCam.Active  then CinematicCam.Stop()     else CinematicCam.Start()     end
			elseif featureId=="OrbitCam"      then if State.OrbitCam        then OrbitCam.Disable()      else OrbitCam.Enable()        end
			elseif featureId=="Respawn"       then doRespawn()
			elseif featureId=="UndoTeleport"  then undoTeleport()
			end
		end
	end
	if not Settings.Keybinds.Fullbright and keyName=="B" then
		if State.Fullbright then Fullbright.Disable() else Fullbright.Enable() end
	end
end)

----------------------------------------------------------------------------
-- RESPONSIVE CLAMPING
----------------------------------------------------------------------------
local function clampWindow()
	local vp   = ScreenGui.AbsoluteSize
	local maxW = math.min(860, vp.X-20)
	local maxH = math.min(540, vp.Y-20)
	if Window.Size.X.Offset > vp.X-10 or Window.Size.Y.Offset > vp.Y-10 then
		Window.Size = UDim2.new(0,maxW,0,maxH)
		fullHeight  = Window.Size
	end
	local pos = Window.AbsolutePosition
	Window.Position = UDim2.new(0,
		math.clamp(pos.X, 0, math.max(0,vp.X-Window.AbsoluteSize.X)),
		0,
		math.clamp(pos.Y, 0, math.max(0,vp.Y-Window.AbsoluteSize.Y)))
end
ScreenGui:GetPropertyChangedSignal("AbsoluteSize"):Connect(clampWindow)
clampWindow()

----------------------------------------------------------------------------
-- ENTRANCE ANIMATION
----------------------------------------------------------------------------
Window.BackgroundTransparency = 1
TitleBar.BackgroundTransparency = 1
local startSize = Window.Size
Window.Size = UDim2.new(0, startSize.X.Offset, 0, 0)
Util.Tween(Window, { Size=startSize, BackgroundTransparency=0 }, 0.4, Enum.EasingStyle.Back)
task.delay(0.18, function() Util.Tween(TitleBar, { BackgroundTransparency=0 }, 0.2) end)

Notify.Push(
	"Admin Panel + Blissful ESP Loaded",
	"Press "..Settings.ToggleKey.." to toggle  ·  Ctrl+P for palette\n"..
	"ESP: Box / Corner / Tracers / Health Bars all ported from Blissful4992/ESPs",
	"Success", 6
)

----------------------------------------------------------------------------
-- APPLY SAVED SETTINGS ON LOAD
----------------------------------------------------------------------------
applyMovementStats()
applyFOV(Settings.FOV)
Overlay.Visible    = Settings.OverlayOn
Workspace.Gravity  = Settings.GravityValue
