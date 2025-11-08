-- LocalScript (StarterPlayerScripts)
local Players = game:GetService("Players")
local UserInput = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local P = Players.LocalPlayer
local PG = P:WaitForChild("PlayerGui")

-- Config
local speed = 0.02 -- segundos entre destaque (diminuir = mais rápido). 0.01 é bem rápido.
local iconSize = 56
local padding = 6
local bgColor = Color3.fromRGB(25,25,25)
local normalAlpha = 0.6
local highlightColor = Color3.fromRGB(255,255,255)
local highlightScale = 1.08 -- escala do ícone ao ser destacado

-- GUI base
local sg = Instance.new("ScreenGui")
sg.Name = "FakeInvHighlighter"
sg.ResetOnSpawn = false
sg.Parent = PG

local bar = Instance.new("Frame", sg)
bar.AnchorPoint = Vector2.new(0.5, 1)
bar.Position = UDim2.new(0.5, 0, 1, -80)
bar.Size = UDim2.new(0, (iconSize+padding)*8 + padding, 0, iconSize + padding*2)
bar.BackgroundTransparency = 1

local bg = Instance.new("Frame", bar)
bg.Size = UDim2.new(1,0,1,0)
bg.Position = UDim2.new(0,0,0,0)
bg.BackgroundColor3 = bgColor
bg.BackgroundTransparency = 1 - normalAlpha
bg.BorderSizePixel = 0
bg.AnchorPoint = Vector2.new(0,0)
bg.ZIndex = 1

local layout = Instance.new("UIListLayout", bar)
layout.Padding = UDim.new(0, padding)
layout.FillDirection = Enum.FillDirection.Horizontal
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.HorizontalAlignment = Enum.HorizontalAlignment.Left

-- store buttons
local iconButtons = {}

local function collectTools()
	local t = {}
	if P.Backpack then
		for _,v in ipairs(P.Backpack:GetChildren()) do
			if v:IsA("Tool") then table.insert(t,v) end
		end
	end
	if P.Character then
		for _,v in ipairs(P.Character:GetChildren()) do
			if v:IsA("Tool") then table.insert(t,v) end
		end
	end
	return t
end

local function rebuildBar()
	-- limpa antigos
	for _,c in ipairs(bar:GetChildren()) do
		if c ~= bg and c ~= layout then
			c:Destroy()
		end
	end
	iconButtons = {}

	local tools = collectTools()
	for i,tool in ipairs(tools) do
		local btn = Instance.new("ImageButton", bar)
		btn.Size = UDim2.new(0, iconSize, 0, iconSize)
		btn.LayoutOrder = i
		btn.AutoButtonColor = false
		btn.BorderSizePixel = 0
		btn.BackgroundTransparency = 0.4
		btn.BackgroundColor3 = Color3.fromHSV((i*0.12)%1,0.6,0.9)
		btn.ZIndex = 2
		-- tenta usar propriedades de ícone comuns (Icon, TextureId, etc)
		local image = ""
		if tool:FindFirstChild("TextureId") and type(tool.TextureId)=="string" then image = tool.TextureId
		elseif tool:FindFirstChild("Icon") and type(tool.Icon)=="string" then image = tool.Icon
		elseif tool:FindFirstChild("Handle") and tool.Handle:FindFirstChild("Texture") then image = tool.Handle.Texture end
		if image and image ~= "" then
			btn.Image = image
		else
			btn.Image = ""
		end
		-- label pequeno opcional
		btn.Name = tool.Name
		table.insert(iconButtons, btn)
	end
end

-- reconstrói quando mudar
if P:FindFirstChild("Backpack") then
	P.Backpack.ChildAdded:Connect(rebuildBar)
	P.Backpack.ChildRemoved:Connect(rebuildBar)
end
P.CharacterAdded:Connect(function(ch)
	ch.ChildAdded:Connect(rebuildBar)
	ch.ChildRemoved:Connect(rebuildBar)
end)
rebuildBar()

-- loop rápido que destaca cada ícone em sequência (apenas visual)
local running = true
spawn(function()
	while true do
		local n = #iconButtons
		if running and n>0 then
			for i = 1, n do
				if not running then break end
				for j,btn in ipairs(iconButtons) do
					-- reset visual
					btn.Size = UDim2.new(0, iconSize, 0, iconSize)
					btn.BackgroundTransparency = 0.4
				end
				local sel = iconButtons[i]
				if sel and sel.Parent then
					-- destaque visual simples: remove transparência e escala
					sel.BackgroundTransparency = 0
					sel.Size = UDim2.new(0, iconSize*highlightScale, 0, iconSize*highlightScale)
					-- espera o tempo configurado
					wait(speed)
				else
					-- se botão inválido, pequeno atraso
					wait(0.01)
				end
			end
		else
			wait(0.05)
		end
	end
end)

-- toggle com tecla T
UserInput.InputBegan:Connect(function(inp, gp)
	if gp then return end
	if inp.KeyCode == Enum.KeyCode.T then
		running = not running
	end
end)
