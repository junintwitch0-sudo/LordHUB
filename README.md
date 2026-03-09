-- LORDHUB v1.0 - Fuja do Tsunami pra Brainrots
-- Base: Rayfield UI
-- Features: Autofarm (osakatp2), Godmode (vinz hub), e muito mais!
-- Desenvolvido para máxima eficiência no jogo

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
	Name = "LORDHUB v1.0",
	LoadingTitle = "Carregando LORDHUB...",
	LoadingSubtitle = "Fuja do Tsunami pra Brainrots",
	ConfigurationSaving = {
		Enabled = true,
		FolderName = "LORDHUB",
		FileName = "Config"
	},
	Discord = {
		Enabled = false,
		Invite = "noinvitelink",
		RememberJoins = true
	},
	KeySystem = false,
	KeySettings = {
		Title = "LORDHUB Key System",
		Subtitle = "Key System",
		Note = "No key required",
		FileName = "LORDHUBKey",
		SaveKey = true,
		SaveKeyFile = true,
		Function = function(Key)
			if Key == "Correct" then
				Rayfield:Notify({
					Title = "Sucesso!",
					Content = "Chave correta!",
					Duration = 6.5,
					Image = 4483362458,
				})
			else
				Rayfield:Notify({
					Title = "Erro!",
					Content = "Chave incorreta!",
					Duration = 6.5,
					Image = 4483362458,
				})
			end
		end
	}
})

-- ============================================
-- VARIÁVEIS GLOBAIS
-- ============================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")

	-- Estados das features
	local Features = {
		AutoFarm = false,
		Godmode = false,
		SpeedBoost = false,
		JumpBoost = false,
		NoClip = false
	}

	-- Variáveis de controle
	local AutoFarmConnection = nil
	local GodmodeConnection = nil
	local SpeedConnection = nil
	local JumpConnection = nil
	local NoClipConnection = nil
	local BrainrotsColetados = 0
	local LimiteBrainrots = 6

-- ============================================
-- FUNÇÕES AUXILIARES
-- ============================================

local function GetCharacter()
	return LocalPlayer.Character
end

local function GetHumanoid()
	local char = GetCharacter()
	if char then
		return char:FindFirstChild("Humanoid")
	end
	return nil
end

local function GetHumanoidRootPart()
	local char = GetCharacter()
	if char then
		return char:FindFirstChild("HumanoidRootPart")
	end
	return nil
end

local function TeleportTo(Position)
	local HRP = GetHumanoidRootPart()
	if HRP then
		HRP.CFrame = CFrame.new(Position)
	end
end

local function GetAllPlayers()
	local PlayerList = {}
	for _, Player in pairs(Players:GetPlayers()) do
		if Player ~= LocalPlayer then
			table.insert(PlayerList, Player)
		end
	end
	return PlayerList
end

local function GetNearestPlayer()
	local NearestPlayer = nil
	local NearestDistance = math.huge
	local HRP = GetHumanoidRootPart()
	
	if not HRP then return nil end
	
	for _, Player in pairs(GetAllPlayers()) do
		local PlayerChar = Player.Character
		if PlayerChar then
			local PlayerHRP = PlayerChar:FindFirstChild("HumanoidRootPart")
			if PlayerHRP then
				local Distance = (HRP.Position - PlayerHRP.Position).Magnitude
				if Distance < NearestDistance then
					NearestDistance = Distance
					NearestPlayer = Player
				end
			end
		end
	end
	
	return NearestPlayer
end

local function DamagePlayer(Player, Damage)
	local PlayerHumanoid = Player.Character:FindFirstChild("Humanoid")
	if PlayerHumanoid then
		PlayerHumanoid:TakeDamage(Damage)
	end
end

-- ============================================
-- AUTOFARM (Baseado em osakatp2)
-- ============================================

	local function IsBrainrotValido(Brainrot)
		local Name = Brainrot.Name:lower()
		-- Verifica se é Celestial, Divino ou Infinity
		if Name:find("celestial") or Name:find("divino") or Name:find("infinity") then
			return true
		end
		return false
	end
	
	local function StartAutoFarm()
		if AutoFarmConnection then
			AutoFarmConnection:Disconnect()
		end
		
		BrainrotsColetados = 0 -- Reset contador
		
		AutoFarmConnection = RunService.Heartbeat:Connect(function()
			if not Features.AutoFarm then return end
			
			-- Se atingiu o limite, volta para safezone
			if BrainrotsColetados >= LimiteBrainrots then
				local HRP = GetHumanoidRootPart()
				if HRP then
					local Workspace = game:GetService("Workspace")
					local SafeZone = Workspace:FindFirstChild("SafeZone") or Workspace:FindFirstChild("Spawn")
					if SafeZone then
						local SafeZonePos = SafeZone:IsA("BasePart") and SafeZone.Position or SafeZone:FindFirstChild("HumanoidRootPart") and SafeZone:FindFirstChild("HumanoidRootPart").Position
					if SafeZonePos then
							local Direction = (SafeZonePos - HRP.Position).Unit
							HRP.CFrame = HRP.CFrame + Direction * 5
						end
					end
				end
				return
			end
			
			local HRP = GetHumanoidRootPart()
			if not HRP then return end
			
			local Workspace = game:GetService("Workspace")
			
			-- Procura por brainrots (modelos/personagens de brainrot)
			local BrainrotFolder = Workspace:FindFirstChild("Brainrots") or Workspace:FindFirstChild("brainrots")
			
			if BrainrotFolder then
				local NearestBrainrot = nil
				local NearestDistance = math.huge
				local NearestBrainrotName = ""
				
				-- Encontra o brainrot válido mais próximo (Celestial, Divino, Infinity)
				for _, Brainrot in pairs(BrainrotFolder:GetChildren()) do
					if IsBrainrotValido(Brainrot) then
						if Brainrot:IsA("Model") or Brainrot:IsA("BasePart") then
							local BrainrotHRP = Brainrot:FindFirstChild("HumanoidRootPart") or Brainrot
							if BrainrotHRP then
								local Distance = (HRP.Position - BrainrotHRP.Position).Magnitude
								if Distance < NearestDistance then
									NearestDistance = Distance
									NearestBrainrot = BrainrotHRP
									NearestBrainrotName = Brainrot.Name
								end
							end
						end
					end
				end
				
					-- Se encontrou um brainrot válido próximo, move para ele
				if NearestBrainrot and NearestDistance < 100 then
					local Direction = (NearestBrainrot.Position - HRP.Position).Unit
					HRP.CFrame = HRP.CFrame + Direction * 5
					
					-- Se chegou perto, incrementa o contador
					if NearestDistance < 5 then
						BrainrotsColetados = BrainrotsColetados + 1
						Rayfield:Notify({
							Title = "Brainrot Coletado",
							Content = BrainrotsColetados .. "/" .. LimiteBrainrots .. " - " .. NearestBrainrotName,
							Duration = 2,
							Image = 4483362458,
						})
					end
				else
					-- Se não há brainrot válido próximo, volta para a safezone
					local SafeZone = Workspace:FindFirstChild("SafeZone") or Workspace:FindFirstChild("Spawn")
					if SafeZone then
						local SafeZonePos = SafeZone:IsA("BasePart") and SafeZone.Position or SafeZone:FindFirstChild("HumanoidRootPart") and SafeZone:FindFirstChild("HumanoidRootPart").Position
						if SafeZonePos then
							local Direction = (SafeZonePos - HRP.Position).Unit
							HRP.CFrame = HRP.CFrame + Direction * 5
						end
					end
				end
			end
		end)
	
	Rayfield:Notify({
		Title = "AutoFarm Ativado",
		Content = "AutoFarm iniciado com sucesso!",
		Duration = 4.5,
		Image = 4483362458,
	})
end

	local function StopAutoFarm()
		if AutoFarmConnection then
			AutoFarmConnection:Disconnect()
			AutoFarmConnection = nil
		end
		
		Rayfield:Notify({
			Title = "AutoFarm Desativado",
			Content = "Coletou " .. BrainrotsColetados .. "/" .. LimiteBrainrots .. " brainrots!",
			Duration = 4.5,
			Image = 4483362458,
		})
	end

-- ============================================
-- GODMODE (Baseado em vinz hub)
-- ============================================

	local function StartGodmode()
		if GodmodeConnection then
			GodmodeConnection:Disconnect()
		end
		
		GodmodeConnection = RunService.Heartbeat:Connect(function()
			if not Features.Godmode then return end
			
			local Character = GetCharacter()
			local Humanoid = GetHumanoid()
			
			if Humanoid and Character then
				-- Aumenta a vida para valores muito altos
				Humanoid.MaxHealth = 999999
				Humanoid.Health = 999999
				
				-- Protege contra dano de ondas/tsunami
				for _, Part in pairs(Character:GetDescendants()) do
					if Part:IsA("BasePart") then
						Part.CanCollide = true
						Part.TopSurface = Enum.SurfaceType.Smooth
						Part.BottomSurface = Enum.SurfaceType.Smooth
					end
				end
			end
		end)
	
	Rayfield:Notify({
		Title = "Godmode Ativado",
		Content = "Você é imortal agora!",
		Duration = 4.5,
		Image = 4483362458,
	})
end

	local function StopGodmode()
		if GodmodeConnection then
			GodmodeConnection:Disconnect()
			GodmodeConnection = nil
		end
		
		local Humanoid = GetHumanoid()
		if Humanoid then
			Humanoid.MaxHealth = 100
			Humanoid.Health = 100
		end
		
		Rayfield:Notify({
			Title = "Godmode Desativado",
			Content = "Godmode foi desligado!",
			Duration = 4.5,
			Image = 4483362458,
		})
	end

-- ============================================
-- SPEED BOOST
-- ============================================

	local function StartSpeedBoost(SpeedAmount)
		if SpeedConnection then
			SpeedConnection:Disconnect()
		end
		
		SpeedConnection = RunService.Heartbeat:Connect(function()
			if not Features.SpeedBoost then return end
			
			local Humanoid = GetHumanoid()
			if Humanoid then
				Humanoid.WalkSpeed = SpeedAmount
			end
		end)
	
	Rayfield:Notify({
		Title = "Speed Boost Ativado",
		Content = "Velocidade aumentada para " .. SpeedAmount,
		Duration = 4.5,
		Image = 4483362458,
	})
end

	local function StopSpeedBoost()
		if SpeedConnection then
			SpeedConnection:Disconnect()
			SpeedConnection = nil
		end
		
		local Humanoid = GetHumanoid()
		if Humanoid then
			Humanoid.WalkSpeed = 16 -- Velocidade padrão
		end
	end

-- ============================================
-- JUMP BOOST
-- ============================================

local function StartJumpBoost(JumpAmount)
	if JumpConnection then
		JumpConnection:Disconnect()
	end
	
	JumpConnection = RunService.Heartbeat:Connect(function()
		if not Features.JumpBoost then return end
		
		local Humanoid = GetHumanoid()
		if Humanoid then
			Humanoid.JumpPower = JumpAmount
		end
	end)
	
	Rayfield:Notify({
		Title = "Jump Boost Ativado",
		Content = "Pulo aumentado para " .. JumpAmount,
		Duration = 4.5,
		Image = 4483362458,
	})
end

local function StopJumpBoost()
	if JumpConnection then
		JumpConnection:Disconnect()
		JumpConnection = nil
	end
	
	local Humanoid = GetHumanoid()
	if Humanoid then
		Humanoid.JumpPower = 50
	end
end

-- ============================================
-- NOCLIP
-- ============================================

local function StartNoClip()
	if NoClipConnection then
		NoClipConnection:Disconnect()
	end
	
	NoClipConnection = RunService.Heartbeat:Connect(function()
		if not Features.NoClip then return end
		
		local Character = GetCharacter()
		if Character then
			for _, Part in pairs(Character:GetDescendants()) do
				if Part:IsA("BasePart") then
					Part.CanCollide = false
				end
			end
		end
	end)
	
	Rayfield:Notify({
		Title = "NoClip Ativado",
		Content = "Você pode atravessar paredes!",
		Duration = 4.5,
		Image = 4483362458,
	})
end

local function StopNoClip()
	if NoClipConnection then
		NoClipConnection:Disconnect()
		NoClipConnection = nil
	end
	
	local Character = GetCharacter()
	if Character then
		for _, Part in pairs(Character:GetDescendants()) do
			if Part:IsA("BasePart") then
				Part.CanCollide = true
			end
		end
	end
end

-- ============================================
-- TELEPORT TO PLAYERS
-- ============================================

local function TeleportToNearestPlayer()
	local NearestPlayer = GetNearestPlayer()
	if NearestPlayer then
		local PlayerChar = NearestPlayer.Character
		if PlayerChar then
			local PlayerHRP = PlayerChar:FindFirstChild("HumanoidRootPart")
			if PlayerHRP then
				TeleportTo(PlayerHRP.Position + Vector3.new(0, 5, 0))
				Rayfield:Notify({
					Title = "Teleportado",
					Content = "Teleportado para " .. NearestPlayer.Name,
					Duration = 4.5,
					Image = 4483362458,
				})
			end
		end
	end
end

-- ============================================
-- TABS E INTERFACE
-- ============================================

-- TAB: COMBAT
local CombatTab = Window:CreateTab("⚔️ Combat", 4483362458)

CombatTab:CreateToggle({
	Name = "AutoFarm",
	Callback = function(Value)
		Features.AutoFarm = Value
		if Value then
			StartAutoFarm()
		else
			StopAutoFarm()
		end
	end,
})

CombatTab:CreateToggle({
	Name = "Godmode",
	Callback = function(Value)
		Features.Godmode = Value
		if Value then
			StartGodmode()
		else
			StopGodmode()
		end
	end,
})



-- TAB: PLAYER
local PlayerTab = Window:CreateTab("👤 Player", 4483362458)

PlayerTab:CreateSlider({
	Name = "Speed Boost",
	Min = 16,
	Max = 2000,
	Default = 16,
	Color = Color3.fromRGB(255, 107, 53),
	Increment = 1,
	Suffix = " Speed",
	Callback = function(Value)
		if Features.SpeedBoost then
			StartSpeedBoost(Value)
		end
	end,
})

PlayerTab:CreateToggle({
	Name = "Enable Speed Boost",
	Callback = function(Value)
		Features.SpeedBoost = Value
		if Value then
			StartSpeedBoost(50)
		else
			StopSpeedBoost()
		end
	end,
})

PlayerTab:CreateSlider({
	Name = "Jump Boost",
	Min = 50,
	Max = 2000,
	Default = 50,
	Color = Color3.fromRGB(255, 107, 53),
	Increment = 10,
	Suffix = " Jump",
	Callback = function(Value)
		if Features.JumpBoost then
			StartJumpBoost(Value)
		end
	end,
})

PlayerTab:CreateToggle({
	Name = "Enable Jump Boost",
	Callback = function(Value)
		Features.JumpBoost = Value
		if Value then
			StartJumpBoost(100)
		else
			StopJumpBoost()
		end
	end,
})

PlayerTab:CreateToggle({
	Name = "NoClip",
	Callback = function(Value)
		Features.NoClip = Value
		if Value then
			StartNoClip()
		else
			StopNoClip()
		end
	end,
})

-- TAB: TELEPORT
local TeleportTab = Window:CreateTab("🌐 Teleport", 4483362458)

TeleportTab:CreateButton({
	Name = "Teleport to Nearest Player",
	Callback = function()
		TeleportToNearestPlayer()
	end,
})

TeleportTab:CreateButton({
	Name = "Teleport to Spawn",
	Callback = function()
		TeleportTo(Vector3.new(0, 50, 0))
		Rayfield:Notify({
			Title = "Teleportado",
			Content = "Teleportado para o spawn!",
			Duration = 4.5,
			Image = 4483362458,
		})
	end,
})

-- TAB: SETTINGS
local SettingsTab = Window:CreateTab("⚙️ Settings", 4483362458)

SettingsTab:CreateLabel("LORDHUB v1.0")
SettingsTab:CreateLabel("Fuja do Tsunami pra Brainrots")
SettingsTab:CreateLabel("Desenvolvido por LORDHUB")
SettingsTab:CreateLabel("")

SettingsTab:CreateButton({
	Name = "Desativar Todas as Features",
	Callback = function()
		Features.AutoFarm = false
		Features.Godmode = false
		Features.SpeedBoost = false
		Features.JumpBoost = false
		Features.NoClip = false
		
		StopAutoFarm()
		StopGodmode()
		StopSpeedBoost()
		StopJumpBoost()
		StopNoClip()
		
		Rayfield:Notify({
			Title = "Todas as Features Desativadas",
			Content = "Todas as features foram desligadas!",
			Duration = 4.5,
			Image = 4483362458,
		})
	end,
})

SettingsTab:CreateButton({
	Name = "Destruir Interface",
	Callback = function()
		Window:Close()
	end,
})

-- ============================================
-- CLEANUP
-- ============================================

LocalPlayer.CharacterAdded:Connect(function(NewCharacter)
	Character = NewCharacter
	Humanoid = Character:WaitForChild("Humanoid")
	
	-- Reaplica features ativas após respawn
	if Features.Godmode then
		StartGodmode()
	end
	if Features.SpeedBoost then
		StartSpeedBoost(50)
	end
	if Features.JumpBoost then
		StartJumpBoost(100)
	end
	if Features.NoClip then
		StartNoClip()
	end
end)

Rayfield:SetNotification({
	Title = "LORDHUB v1.0 Carregado",
	Content = "Script carregado com sucesso! Aproveite!",
	Duration = 6.5,
	Image = 4483362458,
})

print("✅ LORDHUB v1.0 - Script carregado com sucesso!")
