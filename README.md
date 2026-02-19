-- DUMP

-- ORION LIB
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/duysira5/Hehe/refs/heads/main/Orion.lua.txt"))()


local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local LP = Players.LocalPlayer

--================ WINDOW =================
local Window = OrionLib:MakeWindow({
    Name = "Tét Hụb",
    SaveConfig = true,
    ConfigFolder = "FakeHub"
})

local Tab = Window:MakeTab({
    Name = "Main",
    Icon = "rbxassetid://4483345998"
})
local Tab2 = Window:MakeTab({
    Name = "LocalPlayer",
    Icon = "rbxassetid://4483345998"
})
local Tab3 = Window:MakeTab({
    Name = "World",
    Icon = "rbxassetid://4483345998"
})


--// SERVICES
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local lp = Players.LocalPlayer

--// GLOBAL
getgenv().AutoFarm = false
getgenv().SeaSelect = "Sea 1"

--// SEA 1 FULL
local Sea1Table = {
    {1,"Boss","Quest1"},
    {50,"SnowBoss","Quest2"},
    {100,"Sand","Quest3"},
    {250,"Enel","Quest4"},
    {550,"Ace","Quest5"},
    {900,"Minmama","Quest6"},
    {1350,"Dark","Quest7"},
}

--// SEA 2 FULL
local Sea2Table = {
    {2000,"Paw","Quest1"},
    {4000,"Magma Admiral","Quest2"},
    {8000,"Nickbeo","Quest3"},
    {16000,"Tango","Quest4"},
    {20000,"Garou","Quest5"},
    {50000,"Sukuna","Quest6"},
    {100000,"Grab","Quest7"},
      {500000,"Pride","Quest9"},
}

--// LEVEL
local function getLevel()
	local d = lp:FindFirstChild("Data")
	if d and d:FindFirstChild("Levels") then
		return d.Levels.Value
	end
	return 1
end

--// TARGET
local function getTarget()
	local level = getLevel()
	local tableUse = getgenv().SeaSelect == "Sea 1" and Sea1Table or Sea2Table
	local result = tableUse[1]

	for _,v in ipairs(tableUse) do
		if level >= v[1] then
			result = v
		else
			break
		end
	end

	return result[2], result[3]
end

--// FIND NPC
local function findBoss(name)
	local folder = workspace:FindFirstChild("NPC DAMAGE")
	if not folder then return end

	name = string.lower(name)

	for _,npc in pairs(folder:GetChildren()) do
		if string.find(string.lower(npc.Name), name) then
			return npc
		end
	end
end

--// QUEST SMART
local function doQuest(questName)
	local npcs = workspace:FindFirstChild("NPCS")
	if not npcs then return end

	local q = npcs:FindFirstChild(questName)
	if not q then return end

	local qt = q.ClickPart.QuestTake.QuestTake

	pcall(function()
		if getgenv().SeaSelect == "Sea 2" and qt:FindFirstChild("Accept3") then
			qt.Accept3.RemoteEvent:FireServer()
		elseif qt:FindFirstChild("Accept2") then
			qt.Accept2.RemoteEvent:FireServer()
		elseif qt:FindFirstChild("Accept1") then
			qt.Accept1.RemoteEvent:FireServer()
		end
	end)
end

--// TWEEN BEHIND
local function tweenBehind(boss)
	local char = lp.Character
	if not char then return end

	local hrp = char:FindFirstChild("HumanoidRootPart")
	local bhrp = boss and boss:FindFirstChild("HumanoidRootPart")

	if not hrp or not bhrp then return end

	local pos = bhrp.CFrame * CFrame.new(0,0,5)

	TweenService:Create(hrp,TweenInfo.new(0.4),{CFrame=pos}):Play()
end

--// ✅ FIX TOOL REMOTE
local DeadCache = {}

local function fireToolsIfDead(boss)
	if not boss then return end

	local hum = boss:FindFirstChildOfClass("Humanoid")

	local dead = false

	if not hum then dead = true
	elseif hum.Health <= 0 then dead = true
	elseif not boss.Parent then dead = true
	end

	if dead and not DeadCache[boss] then
		DeadCache[boss] = true

		for _,tool in pairs(lp.Backpack:GetChildren()) do
			if tool:IsA("Tool") then
				for _,v in pairs(tool:GetDescendants()) do
					if v:IsA("RemoteEvent") then
						pcall(function()
							v:FireServer()
						end)
					end
				end
			end
		end
	elseif not dead then
		DeadCache[boss] = nil
	end
end

--// LOOP
task.spawn(function()
	while true do
		if getgenv().AutoFarm then

			local bossName, questName = getTarget()
			local boss = findBoss(bossName)

			doQuest(questName)

			if boss then
				tweenBehind(boss)
				fireToolsIfDead(boss)
			end
		end
		task.wait(0.25)
	end
end)

--// UI

Tab:AddDropdown({
	Name = "Select Sea",
	Options = {"Sea 1","Sea 2"},
	Default = "Sea 1",
	Callback = function(v)
		getgenv().SeaSelect = v
	end
})

Tab:AddToggle({
	Name = "Auto Farm",
	Default = false,
	Callback = function(v)
		getgenv().AutoFarm = v
	end
})

OrionLib:Init()


--================ GLOBAL =================
getgenv().SelectedNPC = nil
getgenv().AutoBring = false
getgenv().BringDist = 6

getgenv().AutoHitbox = false
getgenv().HitboxSize = 30

getgenv().OneShot = false
getgenv().AutoOneShot = false

getgenv().SelectedSkill = "Z"
getgenv().AutoSkill = false
getgenv().AutoKill = false

local ShotDelay = 0.4
local LastShot = 0
local HitboxCache = {}

--================ UTILS ==================
local function GetNPC()
    if not getgenv().SelectedNPC then return end
    return workspace["NPC DAMAGE"]:FindFirstChild(getgenv().SelectedNPC)
end

local function GetHRP(model)
    if not model then return end
    return model:FindFirstChild("HumanoidRootPart") or model.PrimaryPart
end

--================ NPC LIST ===============
local NPC_List = {}
pcall(function()
    for _,v in pairs(workspace["NPC DAMAGE"]:GetChildren()) do
        if v:IsA("Model") then
            table.insert(NPC_List, v.Name)
        end
    end
end)

--================ UI =====================
Tab:AddDropdown({
    Name = "Select NPC / Boss",
    Options = NPC_List,
    Callback = function(v)
        getgenv().SelectedNPC = v
    end
})

Tab:AddButton({
    Name = "Teleport To NPC",
    Callback = function()
        local npc = GetNPC()
        local hrp = GetHRP(npc)
        local char = LP.Character
        if hrp and char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = hrp.CFrame * CFrame.new(0,0,-4)
        end
    end
})

--================ BRING ==================
Tab:AddSlider({
    Name = "Bring Distance",
    Min = 3, Max = 15, Default = 6,
    Callback = function(v)
        getgenv().BringDist = v
    end
})

Tab:AddToggle({
    Name = "Auto Bring (Front)",
    Callback = function(v)
        getgenv().AutoBring = v
    end
})

--================ HITBOX =================
Tab:AddSlider({
    Name = "Hitbox Size",
    Min = 10, Max = 80, Default = 30,
    Callback = function(v)
        getgenv().HitboxSize = v
    end
})

Tab:AddToggle({
    Name = "Hitbox Resize",
    Callback = function(v)
        getgenv().AutoHitbox = v
    end
})

--================ ONE SHOT ===============
local function KillNPC()
    local npc = GetNPC()
    if not npc then return end
    local hum = npc:FindFirstChildOfClass("Humanoid")
    if hum and hum.Health > 0 then
        sethiddenproperty(LP,"SimulationRadius",math.huge)
        sethiddenproperty(LP,"MaxSimulationRadius",math.huge)
        hum.Health = 0
    end
end

Tab:AddButton({
    Name = "One Shot",
    Callback = function()
        KillNPC()
    end
})

Tab:AddToggle({
    Name = "Auto One Shot",
    Callback = function(v)
        getgenv().AutoOneShot = v
    end
})

--================ SKILL ==================
Tab:AddDropdown({
    Name = "Select Skill",
    Options = {"Z","X","C","V","B","N","F","K"},
    Default = "Z",
    Callback = function(v)
        getgenv().SelectedSkill = v
    end
})

-- biến
local AutoQuest = false
local SelectedQuest = nil

-- bảng quest
local QuestTable = {
    ["Okaku"] = function()
        workspace.NPCS.Quest10.ClickPart.QuestTake.QuestTake.Accept3.RemoteEvent:FireServer()
    end,

    ["Pride"] = function()
        workspace.NPCS.Quest9.ClickPart.QuestTake.QuestTake.Accept3.RemoteEvent:FireServer()
    end,

    ["Grab"] = function()
        workspace.NPCS.Quest7.ClickPart.QuestTake.QuestTake.Accept2.RemoteEvent:FireServer()
    end,

    ["Sukuna"] = function()
        workspace.NPCS.Quest6.ClickPart.QuestTake.QuestTake.Accept1.RemoteEvent:FireServer()
    end,

    ["Geto"] = function()
        workspace.NPCS.Quest8.ClickPart.QuestTake.QuestTake.Accept1.RemoteEvent:FireServer()
    end,

    ["Cid"] = function()
        workspace.NPCS.Quest8.ClickPart.QuestTake.QuestTake.Accept2.RemoteEvent:FireServer()
    end
}

-- Dropdown chọn quest
Tab:AddDropdown({
    Name = "Chọn Quest",
    Default = "",
    Options = {"Pride", "Grab", "Sukuna", "Geto", "Cid", "Okaku"},
    Callback = function(Value)
        SelectedQuest = Value
    end
})

-- Toggle Auto Quest
Tab:AddToggle({
    Name = "Auto Quest",
    Default = false,
    Callback = function(Value)
        AutoQuest = Value

        if Value then
            task.spawn(function()
                while AutoQuest do
                    if SelectedQuest and QuestTable[SelectedQuest] then
                        pcall(function()
                            QuestTable[SelectedQuest]()
                        end)
                    end
                    task.wait(0.1)
                end
            end)
        end
    end
})

-- Notify
OrionLib:Init()


Tab:AddToggle({
    Name = "Auto Skill When Boss Die",
    Callback = function(v)
        getgenv().AutoSkill = v
    end
})

Tab:AddToggle({
    Name = "Auto Kill (Tween + Fire)",
    Callback = function(v)
        getgenv().AutoKill = v
    end
})

--================ FIRE ===================
local function FireSkill()
    for _,tool in pairs(LP.Backpack:GetChildren()) do
        if tool:IsA("Tool") then
            local s = tool:FindFirstChild(getgenv().SelectedSkill)
            if s and s:FindFirstChild("Fire") then
                pcall(function()
                    s.Fire:FireServer()
                end)
            end
        end
    end
end

local function FireAllRemote()
    for _,tool in pairs(LP.Backpack:GetChildren()) do
        if tool:IsA("Tool") then
            for _,v in pairs(tool:GetDescendants()) do
                if v:IsA("RemoteEvent") then
                    pcall(function()
                        v:FireServer()
                        task.wait(0.05)
                    end)
                end
            end
        end
    end
end

--================ LOOP ===================
RunService.Heartbeat:Connect(function()
    local npc = GetNPC()
    local hrp = GetHRP(npc)
    local char = LP.Character
    local hum = npc and npc:FindFirstChildOfClass("Humanoid")
    if not npc or not hrp or not char or not char:FindFirstChild("HumanoidRootPart") then return end

    -- HITBOX FIX
    if getgenv().AutoHitbox then
        if not HitboxCache[hrp] then
            HitboxCache[hrp] = hrp.Size
        end
        hrp.Size = Vector3.new(getgenv().HitboxSize,getgenv().HitboxSize,getgenv().HitboxSize)
        hrp.CanCollide = false
    elseif HitboxCache[hrp] then
        hrp.Size = HitboxCache[hrp]
    end

    -- BRING
    if getgenv().AutoBring then
        hrp.CFrame =
            char.HumanoidRootPart.CFrame
            + char.HumanoidRootPart.CFrame.LookVector * getgenv().BringDist
    end

    -- AUTO ONE SHOT (ANTI LAG)
    if getgenv().AutoOneShot and tick() - LastShot >= ShotDelay then
        LastShot = tick()
        KillNPC()
    end

    -- AUTO SKILL WHEN DIE
    if getgenv().AutoSkill and hum and hum.Health <= 0 then
        FireSkill()
    end

    -- AUTO KILL
    if getgenv().AutoKill then
        TweenService:Create(
            char.HumanoidRootPart,
            TweenInfo.new(0.25,Enum.EasingStyle.Linear),
            {CFrame = hrp.CFrame * CFrame.new(0,0,4)}
        ):Play()
        FireAllRemote()
    end
end)

OrionLib:Init()
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "Toggleui"
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

local Toggle = Instance.new("TextButton")
Toggle.Name = "Toggle"
Toggle.Parent = ScreenGui
Toggle.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Toggle.BackgroundTransparency = 0.5
Toggle.Position = UDim2.new(0, 0, 0.454706937, 0)
Toggle.Size = UDim2.new(0, 50, 0, 50)
Toggle.Draggable = true

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0.2, 0)
Corner.Parent = Toggle

local Image = Instance.new("ImageLabel")
Image.Name = "Icon"
Image.Parent = Toggle
Image.Size = UDim2.new(1, 0, 1, 0)
Image.BackgroundTransparency = 1
Image.Image = "rbxassetid://259226716" 

local Corner2 = Instance.new("UICorner")
Corner2.CornerRadius = UDim.new(0.2, 0)
Corner2.Parent = Image

Toggle.MouseButton1Click:Connect(function()
  OrionLib:ToggleUi()
end)
local Player = game.Players.LocalPlayer
local Char = Player.Character or Player.CharacterAdded:Wait()
local HRP = Char:WaitForChild("HumanoidRootPart")

Player.CharacterAdded:Connect(function(c)
    Char = c
    HRP = Char:WaitForChild("HumanoidRootPart")
end)

--// Biến
local AuraEnabled = false
local AuraSize = 15
local AuraBall

--// Tạo Aura
local function CreateAura()

	if AuraBall then AuraBall:Destroy() end

	AuraBall = Instance.new("Part")
	AuraBall.Shape = Enum.PartType.Ball
	AuraBall.Material = Enum.Material.Neon
	AuraBall.Color = Color3.fromRGB(255,0,0)
	AuraBall.Transparency = 0.3

	AuraBall.Anchored = true -- QUAN TRỌNG
	AuraBall.CanCollide = false
	AuraBall.CanTouch = true
	AuraBall.Massless = true

	AuraBall.Size = Vector3.new(AuraSize,AuraSize,AuraSize)
	AuraBall.Parent = workspace

	-- Follow player mỗi frame
	task.spawn(function()
		while AuraEnabled and AuraBall and HRP do
			AuraBall.CFrame = HRP.CFrame
			task.wait()
		end
	end)

	-- Touch trigger
	AuraBall.Touched:Connect(function(hit)
		local model = hit.Parent
		local plr = game.Players:GetPlayerFromCharacter(model)

		if plr and plr ~= Player then

			-- Backpack tools
			for _,tool in pairs(Player.Backpack:GetChildren()) do
				if tool:IsA("Tool") then
					for _,r in pairs(tool:GetDescendants()) do
						if r:IsA("RemoteEvent") then
							pcall(function()
								r:FireServer()
							end)
						end
					end
				end
			end

			-- Tool đang cầm
			local equip = Char:FindFirstChildOfClass("Tool")
			if equip then
				for _,r in pairs(equip:GetDescendants()) do
					if r:IsA("RemoteEvent") then
						pcall(function()
							r:FireServer()
						end)
					end
				end
			end
		end
	end)
end

--// Toggle
Tab2:AddToggle({
	Name = "Auto Skill Aura",
	Default = false,
	Callback = function(Value)
		AuraEnabled = Value

		if AuraEnabled then
			CreateAura()
		else
			if AuraBall then
				AuraBall:Destroy()
				AuraBall = nil
			end
		end
	end
})

--// Slider vàng
Tab2:AddSlider({
	Name = "Aura Size",
	Min = 10,
	Max = 120,
	Default = 15,
	Color = Color3.fromRGB(255,215,0),
	Increment = 1,
	ValueName = "Studs",
	Callback = function(Value)
		AuraSize = Value
		if AuraBall then
			AuraBall.Size = Vector3.new(Value,Value,Value)
		end
	end
})


-- Label
Tab2:AddLabel("LocalPlayer")

--// Services
local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// ESP STORAGE
local ESPEnabled = false
local BackpackESPEnabled = false
local TracerEnabled = false
local ESPSize = 14

local ESPFolder = Instance.new("Folder", game.CoreGui)
ESPFolder.Name = "ESP_STORAGE"

--================ ESP CREATE =================--

local function CreateESP(Player)
	if Player == LP then return end
	
	local Billboard = Instance.new("BillboardGui")
	Billboard.Size = UDim2.new(0,200,0,50)
	Billboard.AlwaysOnTop = true
	Billboard.Parent = ESPFolder
	
	local Text = Instance.new("TextLabel", Billboard)
	Text.Size = UDim2.new(1,0,1,0)
	Text.BackgroundTransparency = 1
	Text.TextColor3 = Color3.new(1,0,0)
	Text.TextStrokeTransparency = 0
	Text.TextScaled = true
	
	task.spawn(function()
		while ESPEnabled and Player.Parent do
			
			local Char = Player.Character
			if Char and Char:FindFirstChild("Head") then
				
				Billboard.Adornee = Char.Head
				
				local level = "?"
				pcall(function()
					level = Player.Data.Level.Value
				end)
				
				local hp = "?"
				local hum = Char:FindFirstChildOfClass("Humanoid")
				if hum then
					hp = math.floor(hum.Health)
				end
				
				Text.Text = Player.Name.." | Lv:"..level.." | HP:"..hp
				Text.TextSize = ESPSize
				
			end
			
			task.wait(0.2)
		end
		
		Billboard:Destroy()
	end)
end

--================ BACKPACK ESP =================--

local function CreateBackpackESP(Player)
	if Player == LP then return end
	
	local Billboard = Instance.new("BillboardGui")
	Billboard.Size = UDim2.new(0,200,0,40)
	Billboard.AlwaysOnTop = true
	Billboard.Parent = ESPFolder
	
	local Text = Instance.new("TextLabel", Billboard)
	Text.Size = UDim2.new(1,0,1,0)
	Text.BackgroundTransparency = 1
	Text.TextColor3 = Color3.new(0,1,1)
	Text.TextStrokeTransparency = 0
	Text.TextScaled = true
	
	task.spawn(function()
		while BackpackESPEnabled and Player.Parent do
			
			local Char = Player.Character
			if Char and Char:FindFirstChild("Head") then
				
				Billboard.Adornee = Char.Head
				
				local tools = {}
				for _,t in pairs(Player.Backpack:GetChildren()) do
					table.insert(tools, t.Name)
				end
				
				Text.Text = table.concat(tools,", ")
				Text.TextSize = ESPSize
				
			end
			
			task.wait(1)
		end
		
		Billboard:Destroy()
	end)
end

--================ TRACER =================--

local function CreateTracer(Player)
	if Player == LP then return end
	
	local Line = Drawing.new("Line")
	Line.Thickness = 2
	Line.Color = Color3.new(1,1,1)
	
	task.spawn(function()
		while TracerEnabled and Player.Parent do
			
			local Char = Player.Character
			if Char and Char:FindFirstChild("HumanoidRootPart") then
				
				local pos, vis = Camera:WorldToViewportPoint(Char.HumanoidRootPart.Position)
				
				if vis then
					Line.Visible = true
					Line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
					Line.To = Vector2.new(pos.X,pos.Y)
				else
					Line.Visible = false
				end
				
			end
			
			task.wait()
		end
		
		Line:Remove()
	end)
end

--================ TOGGLES =================--

Tab2:AddToggle({
	Name = "ESP Player",
	Default = false,
	Callback = function(v)
		ESPEnabled = v
		
		if v then
			for _,p in pairs(Players:GetPlayers()) do
				CreateESP(p)
			end
		end
	end
})

Tab2:AddToggle({
	Name = "ESP Backpack",
	Default = false,
	Callback = function(v)
		BackpackESPEnabled = v
		
		if v then
			for _,p in pairs(Players:GetPlayers()) do
				CreateBackpackESP(p)
			end
		end
	end
})

Tab2:AddToggle({
	Name = "Tracers",
	Default = false,
	Callback = function(v)
		TracerEnabled = v
		
		if v then
			for _,p in pairs(Players:GetPlayers()) do
				CreateTracer(p)
			end
		end
	end
})

--================ SLIDER =================--

Tab2:AddSlider({
	Name = "ESP Size",
	Min = 10,
	Max = 30,
	Default = 14,
	Increment = 1,
	ValueName = "Size",
	Callback = function(v)
		ESPSize = v
	end
})

--// Services
local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local Char = LP.Character or LP.CharacterAdded:Wait()
local HRP = Char:WaitForChild("HumanoidRootPart")

LP.CharacterAdded:Connect(function(c)
	Char = c
	HRP = c:WaitForChild("HumanoidRootPart")
end)

--==================================================
-- AUTO PLAY
--==================================================

local AutoPlay = false

Tab3:AddToggle({
	Name = "Auto Nhấn Play",
	Default = false,
	Callback = function(v)
		AutoPlay = v
		
		task.spawn(function()
			while AutoPlay do
				pcall(function()
					for _,v in pairs(game:GetDescendants()) do
						if v.Name == "Play" and v:IsA("RemoteEvent") then
							v:FireServer()
						end
					end
				end)
				task.wait(2)
			end
		end)
	end
})

--==================================================
-- AUTO SEA 2
--==================================================

local AutoSea2 = false

Tab3:AddToggle({
	Name = "Auto Qua Sea 2",
	Default = false,
	Callback = function(v)
		AutoSea2 = v
		
		task.spawn(function()
			while AutoSea2 do
				pcall(function()
					for _,v in pairs(game:GetDescendants()) do
						if string.find(v.Name,"Sea") and v:IsA("RemoteEvent") then
							v:FireServer()
						end
					end
				end)
				task.wait(3)
			end
		end)
	end
})

--==================================================
-- KILLAURA
--==================================================

Tab3:AddLabel("Killaura All Mobs")

local KillAura = false
local AuraSize = 30
local AuraCircle

-- tạo vòng dưới chân
local function CreateCircle()

	if AuraCircle then AuraCircle:Destroy() end
	
	AuraCircle = Instance.new("Part")
	AuraCircle.Anchored = true
	AuraCircle.CanCollide = false
	AuraCircle.Transparency = 0.5
	AuraCircle.Material = Enum.Material.Neon
	AuraCircle.Color = Color3.fromRGB(173,261,230)
	AuraCircle.Shape = Enum.PartType.Cylinder
	AuraCircle.Size = Vector3.new(0.2, AuraSize, AuraSize)
	AuraCircle.Parent = workspace
	
	task.spawn(function()
		while KillAura and AuraCircle do
			if HRP then
				AuraCircle.CFrame = HRP.CFrame * CFrame.new(0,-3,0) * CFrame.Angles(0,0,math.rad(90))
			end
			task.wait()
		end
	end)
end

-- Kill Aura loop (OPTIMIZE)
task.spawn(function()

	while task.wait(2.4) do
		
		if KillAura and HRP then
			
			pcall(function()
				sethiddenproperty(LP,"SimulationRadius",9999999)
				sethiddenproperty(LP,"MaxSimulationRadius",9999999)
			end)

			for _,mob in pairs(workspace:GetDescendants()) do
				
				if mob:IsA("Humanoid") then
					local model = mob.Parent
					
					if model and model ~= Char and mob.Health > 0 then
						
						local root = model:FindFirstChild("HumanoidRootPart")
						if root then
							
							local dist = (root.Position - HRP.Position).Magnitude
							
							if dist <= AuraSize then
								mob.Health = 0
							end
							
						end
						
					end
				end
				
			end
			
		end
		
	end
	
end)

-- Toggle
Tab3:AddToggle({
	Name = "Kill Aura",
	Default = false,
	Callback = function(v)
		KillAura = v
		
		if v then
			CreateCircle()
		else
			if AuraCircle then AuraCircle:Destroy() end
		end
	end
})

-- Slider
Tab3:AddSlider({
	Name = "kích thước Aura ",
	Min = 10,
	Max = 520,
	Default = 30,
	Increment = 1,
	ValueName = "Studs",
	Callback = function(v)
		AuraSize = v
		
		if AuraCircle then
			AuraCircle.Size = Vector3.new(0.2, v, v)
		end
	end
})


--------------------------------------------------
-- ⭐ AUTO ARMOR
--------------------------------------------------
getgenv().AutoArmor = false

Tab3:AddToggle({
	Name = "Auto Armor ⭐",
	Default = false,
	Callback = function(v)
		getgenv().AutoArmor = v
	end
})

task.spawn(function()
	while true do
		if getgenv().AutoArmor then
			local Players = game:GetService("Players")
			local armors = {
				"Cursed-Armor","Unique-Armor","Darkness-Armor",
				"Thunder-Armor","Diamond-Armor","Golden-Armor",
				"Epic-Armor","Iron-Armor","Godly-Armor"
			}

			for _, player in pairs(Players:GetPlayers()) do
				local backpack = player:FindFirstChild("Backpack")
				if backpack then
					for _, armorName in pairs(armors) do
						local armor = backpack:FindFirstChild(armorName)
						if armor and armor:FindFirstChild("K") and armor.K:FindFirstChild("Fire") then
							armor.K.Fire:FireServer()
						end
					end
				end
			end
		end
		task.wait(0.1)
	end
end)

--------------------------------------------------
-- ✳️ AUTO CẤT (Auto dùng tool đang cầm)
--------------------------------------------------
getgenv().AutoStore = false

Tab3:AddToggle({
	Name = "Auto Cất ✳️",
	Default = false,
	Callback = function(v)
		getgenv().AutoStore = v
	end
})

task.spawn(function()
	while true do
		if getgenv().AutoStore then
			local char = game.Players.LocalPlayer.Character
			if char then
				for _, tool in pairs(char:GetChildren()) do
					if tool:IsA("Tool") and tool:FindFirstChild("RemoteEvent") then
						tool.RemoteEvent:FireServer()
					end
				end
			end
		end
		task.wait(0.3)
	end
end)

--------------------------------------------------
-- 🦘 INF JUMP
--------------------------------------------------
getgenv().InfJump = false

Tab3:AddToggle({
	Name = "Inf Jump 🦘",
	Default = false,
	Callback = function(v)
		getgenv().InfJump = v
	end
})

game:GetService("UserInputService").JumpRequest:Connect(function()
	if getgenv().InfJump then
		local hum = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
		if hum then
			hum:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

--------------------------------------------------
-- 👻 NOCLIP
--------------------------------------------------
getgenv().Noclip = false

Tab3:AddToggle({
	Name = "Noclip ",
	Default = false,
	Callback = function(v)
		getgenv().Noclip = v
	end
})

task.spawn(function()
	while true do
		if getgenv().Noclip then
			local char = game.Players.LocalPlayer.Character
			if char then
				for _, v in pairs(char:GetDescendants()) do
					if v:IsA("BasePart") then
						v.CanCollide = false
					end
				end
			end
		end
		task.wait()
	end
end)

--------------------------------------------------
-- 💰 NHẶT ALL ITEM BOSS (CLICK DETECTOR)
--------------------------------------------------
getgenv().AutoBossLoot = false

Tab3:AddToggle({
	Name = "Nhặt All Item Boss Delay[1s]",
	Default = false,
	Callback = function(v)
		getgenv().AutoBossLoot = v
	end
})

task.spawn(function()
	while true do
		if getgenv().AutoBossLoot then
			for _, obj in pairs(workspace:GetChildren()) do
				local cd = obj:FindFirstChildOfClass("ClickDetector")
				if cd then
					fireclickdetector(cd)
				end
			end
		end
		task.wait(1)
	end
end)

--------------------------------------------------
-- 🎒 HÚT ITEM MAP → BACKPACK
--------------------------------------------------
getgenv().ItemVacuum = false

Tab3:AddToggle({
	Name = "Hút All Item Map 🎒",
	Default = false,
	Callback = function(v)
		getgenv().ItemVacuum = v
	end
})

task.spawn(function()
	while true do
		if getgenv().ItemVacuum then
			local player = game.Players.LocalPlayer
			local char = player.Character
			if char and char:FindFirstChild("HumanoidRootPart") then
				for _, v in pairs(workspace:GetDescendants()) do
					if v:IsA("Tool") then
						v.Handle.CFrame = char.HumanoidRootPart.CFrame
					end
				end
			end
		end
		task.wait(0.5)
	end
end)
