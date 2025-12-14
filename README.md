local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/tbao143/Library-ui/refs/heads/main/Redzhubui"))()
local Window = redzlib:MakeWindow({
	Title = "🍿PIPOCA HUBV1.0",
	SubTitle = "RODRIGO999",
	SaveFolder = "testando | redz lib v5.lua"
})

local Tab1 = Window:MakeTab({"OPÇÕES", "cherry"})

-- Variáveis usadas no spectate
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local CurrentTarget = nil

-- Função para pegar um jogador aleatório (diferente de você e do último)
local function GetRandomPlayer()
	local all = {}
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Humanoid") then
			table.insert(all, p)
		end
	end
	if #all == 0 then return nil end
	local new
	repeat
		new = all[math.random(1, #all)]
	until new ~= CurrentTarget or #all == 1
	return new
end

-- Toggle principal
local Toggle1 = Tab1:AddToggle({
	Name = "VISUALIZAR JOGADORES",
	Description = "[ATIVE E DESATIVE PARA MUDAR DE ALVO]",
	Default = false,
	Callback = function(Value)
		if Value then
			-- Ativado: escolhe um jogador aleatório
			local target = GetRandomPlayer()
			if target then
				CurrentTarget = target
				Camera.CameraSubject = target.Character:FindFirstChild("Humanoid")
				Camera.CameraType = Enum.CameraType.Custom
				game.StarterGui:SetCore("SendNotification", {
					Title = "🐾 " .. target.Name,
					Text = "",
					Duration = 1.5
				})
			else
				game.StarterGui:SetCore("SendNotification", {
					Title = "🎃NENHUM JOGADOR",
					Text = "",
					Duration = 1.5
				})
			end
		else
			-- Desativado: volta para o seu personagem
			CurrentTarget = nil
			if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
				Camera.CameraSubject = LocalPlayer.Character:FindFirstChild("Humanoid")
				game.StarterGui:SetCore("SendNotification", {
					Title = "👻DESATIVADO",
					Text = "",
					Duration = 1.5
				})
			end
		end
	end
})

-- Resto da sua hub
local Tab1 = Window:MakeTab({"OPÇÕES", "sword"})

Tab1:AddButton({"SCRIPTS", function(Value)
	print("Hello World!")
end})

-- ============================================
-- SISTEMA DE VELOCIDADE
-- ============================================

local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local TextChatService = game:GetService("TextChatService")
local UserInputService = game:GetService("UserInputService")

local player = LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

local speedAtivo = false
local velocidadeAtual = 100

local function definirVelocidade(ativa, valor)
	local character = player.Character
	if not character then return end
	
	local humanoid = character:FindFirstChild("Humanoid")
	if not humanoid then return end
	
	if ativa then
		humanoid.WalkSpeed = valor or velocidadeAtual
		game.StarterGui:SetCore("SendNotification", {
			Title = "VELOCIDADE " .. (valor or velocidadeAtual),
			Text = "",
			Duration = 1.5
		})
	else
		humanoid.WalkSpeed = 16
		game.StarterGui:SetCore("SendNotification", {
			Title = "DESATIVADO",
			Text = "",
			Duration = 1.
		})
	end
end

-- Atualizar velocidade quando reseta
player.CharacterAdded:Connect(function(character)
	wait(0.5)
	if speedAtivo then
		local humanoid = character:WaitForChild("Humanoid")
		if humanoid then
			humanoid.WalkSpeed = velocidadeAtual
		end
	end
	
	-- Recriar vôo se estava ativo
	if flying then
		wait(0.5)
		startFlying()
	end
	
	-- Recriar TP Tool se estava ativo
	if tpToolActive then
		wait(1)
		removeTPTool()
		tpTool = createTPTool()
		local hum = character:FindFirstChild("Humanoid")
		if hum then
			hum:EquipTool(tpTool)
		end
	end
end)

-- Toggle de Velocidade
Tab1:AddToggle({
	Name = "LIGAR VELOCIDADE",
	Default = false,
	Callback = function(v)
		speedAtivo = v
		definirVelocidade(v, velocidadeAtual)
	end
})

-- Comandos de Chat para Speed
local function setupSpeedCommands()
	if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
		TextChatService.SendingMessage:Connect(function(message)
			local msg = string.lower(message.Text)
			
			if msg == "/speed" then
				speedAtivo = true
				task.spawn(function()
					definirVelocidade(true, velocidadeAtual)
				end)
				message.Text = ""
			elseif msg == "/unspeed" then
				speedAtivo = false
				task.spawn(function()
					definirVelocidade(false)
				end)
				message.Text = ""
			elseif msg:match("^/speed%s+") then
				local newSpeed = tonumber(msg:match("^/speed%s+(%d+)"))
				if newSpeed and newSpeed >= 16 and newSpeed <= 500 then
					velocidadeAtual = newSpeed
					speedAtivo = true
					task.spawn(function()
						definirVelocidade(true, velocidadeAtual)
					end)
				end
				message.Text = ""
			end
		end)
	else
		player.Chatted:Connect(function(msg)
			local message = string.lower(msg)
			
			if message == "/speed" then
				speedAtivo = true
				task.spawn(function()
					definirVelocidade(true, velocidadeAtual)
				end)
			elseif message == "/unspeed" then
				speedAtivo = false
				task.spawn(function()
					definirVelocidade(false)
				end)
			elseif message:match("^/speed%s+") then
				local newSpeed = tonumber(message:match("^/speed%s+(%d+)"))
				if newSpeed and newSpeed >= 16 and newSpeed <= 500 then
					velocidadeAtual = newSpeed
					speedAtivo = true
					task.spawn(function()
						definirVelocidade(true, velocidadeAtual)
					end)
				end
			end
		end)
	end
end

setupSpeedCommands()

Tab1:AddButton({"VÔO ADMINISTRATIVO", function(Value)
	print("Hello World!")
end})

Tab1:AddDiscordInvite({
	Name = "PIPOCA DISCORD",
	Description = "Join server",
	Logo = "rbxassetid://136837220916793",
	Invite = "https://discord.gg/6zdZgbhS",
})

-- ============================================
-- SISTEMA DE VÔO ADMINISTRATIVO
-- ============================================

-- Variáveis do vôo
local flying = false
local speed = 100
local connection
local bv, bg
local animationTrack
local animationId = "rbxassetid://507766388"

-- Funções do vôo
local function startFlying()
	if flying then return end
	flying = true
	
	local character = player.Character or player.CharacterAdded:Wait()
	local hrp = character:WaitForChild("HumanoidRootPart")
	local humanoid = character:WaitForChild("Humanoid")

	-- Criar BodyVelocity e BodyGyro
	bv = Instance.new("BodyVelocity")
	bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
	bv.Velocity = Vector3.zero
	bv.P = 1000
	bv.Parent = hrp

	bg = Instance.new("BodyGyro")
	bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
	bg.P = 10000
	bg.CFrame = hrp.CFrame
	bg.Parent = hrp

	-- Tocar animação
	local anim = Instance.new("Animation")
	anim.AnimationId = animationId
	animationTrack = humanoid:LoadAnimation(anim)
	animationTrack.Priority = Enum.AnimationPriority.Action
	animationTrack:Play()

	-- Loop de vôo
	connection = RunService.RenderStepped:Connect(function()
		local direction = camera.CFrame.LookVector
		bv.Velocity = direction * speed

		local tiltAngle = math.rad(30)
		local flatLook = Vector3.new(direction.X, 0, direction.Z).Unit
		local tiltDirection = CFrame.new(hrp.Position, hrp.Position + flatLook) * CFrame.Angles(-tiltAngle, 0, 0)
		bg.CFrame = tiltDirection
	end)
	
	game.StarterGui:SetCore("SendNotification", {
		Title = "VÔO ATIVADO",
		Text = "",
		Duration = 1.5
	})
end

local function stopFlying()
	if not flying then return end
	flying = false
	
	if connection then connection:Disconnect() end
	if bv then bv:Destroy() end
	if bg then bg:Destroy() end
	if animationTrack then
		animationTrack:Stop()
		animationTrack:Destroy()
	end
	
	game.StarterGui:SetCore("SendNotification", {
		Title = "VÔO DESATIVADO",
		Text = "",
		Duration = 1.5
	})
end

local function toggleFly()
	if flying then
		stopFlying()
	else
		startFlying()
	end
end

-- Toggle de Vôo
Tab1:AddToggle({
	Name = "LIGAR E DESLIGAR",
	Default = false,
	Callback = function(v)
		if v then
			startFlying()
		else
			stopFlying()
		end
	end
})

-- Comandos de Chat para Vôo
local function setupChatCommands()
	-- Para TextChatService (novo sistema)
	if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
		TextChatService.SendingMessage:Connect(function(message)
			local msg = string.lower(message.Text)
			
			if msg == ";fly" then
				task.spawn(startFlying)
				message.Text = ""
			elseif msg == ";unfly" then
				task.spawn(stopFlying)
				message.Text = ""
			elseif msg:match("^;speed%s+") then
				local newSpeed = tonumber(msg:match("^;speed%s+(%d+)"))
				if newSpeed and newSpeed >= 10 and newSpeed <= 500 then
					speed = newSpeed
					task.spawn(function()
						game.StarterGui:SetCore("SendNotification", {
							Title = "VELOCIDADE VÔO " .. speed,
							Text = "",
							Duration = 1.5
						})
					end)
				end
				message.Text = ""
			end
		end)
	else
		-- Para LegacyChatService (sistema antigo)
		player.Chatted:Connect(function(msg)
			local message = string.lower(msg)
			
			if message == ";fly" then
				task.spawn(startFlying)
			elseif message == ";unfly" then
				task.spawn(stopFlying)
			elseif message:match("^;speed%s+") then
				local newSpeed = tonumber(message:match("^;speed%s+(%d+)"))
				if newSpeed and newSpeed >= 10 and newSpeed <= 500 then
					speed = newSpeed
					task.spawn(function()
						game.StarterGui:SetCore("SendNotification", {
							Title = "VELOCIDADE VÔO " .. speed,
							Text = "",
							Duration = 1.5
						})
					end)
				end
			end
		end)
	end
end

setupChatCommands()

local Paragraph = Tab1:AddParagraph({"SCRIPT FEITO POR @4K_SMITH2F"})

Tab1:AddButton({"TP TOOL", function(Value)
	print("Hello World!")
end})

-- ============================================
-- SISTEMA DE TP TOOL INTEGRADO
-- ============================================

local tpTool = nil
local tpToolActive = false

-- Função para criar partículas de teleporte
local function createTeleportParticles(position)
	for i = 1, 20 do
		local part = Instance.new("Part")
		part.Size = Vector3.new(0.3, 0.3, 0.3)
		part.Position = position
		part.Anchored = true
		part.CanCollide = false
		part.Material = Enum.Material.Neon
		part.Color = Color3.fromRGB(138, 43, 226)
		part.Parent = workspace
		
		local randomDir = Vector3.new(
			math.random(-10, 10),
			math.random(-10, 10),
			math.random(-10, 10)
		)
		
		local tween = TweenService:Create(part, TweenInfo.new(0.5), {
			Position = position + randomDir,
			Transparency = 1,
			Size = Vector3.new(0, 0, 0)
		})
		tween:Play()
		
		game:GetService("Debris"):AddItem(part, 0.5)
	end
end

-- Função de teleporte com animação
local function teleportPlayer(targetPosition)
	local character = player.Character
	if not character then return end
	
	local hrp = character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	
	-- Partículas na posição atual
	createTeleportParticles(hrp.Position)
	
	-- Flash de saída
	local flash1 = Instance.new("Part")
	flash1.Size = Vector3.new(8, 8, 8)
	flash1.Position = hrp.Position
	flash1.Anchored = true
	flash1.CanCollide = false
	flash1.Shape = Enum.PartType.Ball
	flash1.Material = Enum.Material.Neon
	flash1.Color = Color3.fromRGB(138, 43, 226)
	flash1.Transparency = 0.5
	flash1.Parent = workspace
	
	local tween1 = TweenService:Create(flash1, TweenInfo.new(0.3), {
		Size = Vector3.new(0.1, 0.1, 0.1),
		Transparency = 1
	})
	tween1:Play()
	game:GetService("Debris"):AddItem(flash1, 0.3)
	
	wait(0.3)
	
	-- Teleportar
	hrp.CFrame = CFrame.new(targetPosition + Vector3.new(0, 3, 0))
	
	-- Partículas no destino
	createTeleportParticles(hrp.Position)
	
	-- Flash de chegada
	local flash2 = Instance.new("Part")
	flash2.Size = Vector3.new(0.1, 0.1, 0.1)
	flash2.Position = hrp.Position
	flash2.Anchored = true
	flash2.CanCollide = false
	flash2.Shape = Enum.PartType.Ball
	flash2.Material = Enum.Material.Neon
	flash2.Color = Color3.fromRGB(138, 43, 226)
	flash2.Transparency = 0.5
	flash2.Parent = workspace
	
	local tween2 = TweenService:Create(flash2, TweenInfo.new(0.3), {
		Size = Vector3.new(8, 8, 8),
		Transparency = 1
	})
	tween2:Play()
	game:GetService("Debris"):AddItem(flash2, 0.3)
end

-- Função para criar a TP Tool
local function createTPTool()
	local tool = Instance.new("Tool")
	tool.Name = "TP TOOL"
	tool.RequiresHandle = true
	tool.CanBeDropped = false
	
	-- Handle (cabo da ferramenta)
	local handle = Instance.new("Part")
	handle.Name = "Handle"
	handle.Size = Vector3.new(0.3, 1, 0.3)
	handle.Material = Enum.Material.Neon
	handle.Color = Color3.fromRGB(138, 43, 226)
	handle.Parent = tool
	
	-- Brilho
	local pointLight = Instance.new("PointLight")
	pointLight.Color = Color3.fromRGB(138, 43, 226)
	pointLight.Brightness = 2
	pointLight.Range = 10
	pointLight.Parent = handle
	
	-- Evento de clique
	tool.Activated:Connect(function()
		local target = mouse.Hit.Position
		teleportPlayer(target)
	end)
	
	tool.Parent = player.Backpack
	return tool
end

-- Remover ferramenta
local function removeTPTool()
	if tpTool then
		tpTool:Destroy()
		tpTool = nil
	end
end

-- Função para tocar som
local function playSound(soundId, volume)
	local sound = Instance.new("Sound")
	sound.SoundId = soundId
	sound.Volume = volume or 0.5
	sound.Parent = game:GetService("SoundService")
	sound:Play()
	
	game:GetService("Debris"):AddItem(sound, 3)
end

-- Toggle TP TOOL
Tab1:AddToggle({
	Name = "ATIVAR TP TOOL",
	Default = false,
	Callback = function(v)
		tpToolActive = v
		
		if v then
			-- Ativar TP Tool
			tpTool = createTPTool()
			local character = player.Character
			if character then
				local humanoid = character:FindFirstChild("Humanoid")
				if humanoid then
					humanoid:EquipTool(tpTool)
				end
			end
			
			-- Som de ativação (som de power up)
			playSound("rbxassetid://3398620867", 0.6)
		else
			-- Desativar TP Tool
			removeTPTool()
			
			-- Som de desativação (som de power down)
			playSound("rbxassetid://3398620626", 0.6)
		end
	end
})

-- ============================================
-- ABA EM-BREVE COM ESP INTEGRADO
-- ============================================
local Tab2 = Window:MakeTab({"ESP LINE", "cherry"})
local Paragraph = Tab2:AddParagraph({"👻SCRIPTS ADICIONAIS A BAIXO"})

-- Variáveis do ESP
local espAtivado = false
local objetosESP = {}
local espInicializado = false  -- Variável para controlar primeira execução

-- Função para criar o ESP
local function criarESP(jogador)
	if jogador == LocalPlayer then return end
	local char = jogador.Character or jogador.CharacterAdded:Wait()
	local head = char:WaitForChild("Head", 2)
	if not head then return end

	-- Billboard com nome
	local Billboard = Instance.new("BillboardGui")
	Billboard.Adornee = head
	Billboard.Size = UDim2.new(0, 150, 0, 30)
	Billboard.StudsOffset = Vector3.new(0, 2.5, 0)
	Billboard.AlwaysOnTop = true

	local Nome = Instance.new("TextLabel")
	Nome.Size = UDim2.new(1, 0, 1, 0)
	Nome.BackgroundTransparency = 1
	Nome.Text = jogador.Name
	Nome.TextColor3 = Color3.fromRGB(0, 255, 0)
	Nome.TextStrokeTransparency = 0.7
	Nome.TextScaled = false
	Nome.Font = Enum.Font.SourceSansLight
	Nome.TextSize = 10
	Nome.Parent = Billboard

	Billboard.Parent = char

	objetosESP[jogador] = {
		Billboard = Billboard,
		Line = Drawing.new("Line")
	}

	local line = objetosESP[jogador].Line
	line.Color = Color3.fromRGB(0, 255, 0)
	line.Thickness = 1.3
	line.Transparency = 0.9
	line.Visible = false
end

-- Remover ESP
local function removerESP(jogador)
	if objetosESP[jogador] then
		if objetosESP[jogador].Line then
			objetosESP[jogador].Line:Remove()
		end
		if objetosESP[jogador].Billboard then
			objetosESP[jogador].Billboard:Destroy()
		end
		objetosESP[jogador] = nil
	end
end

-- Atualizar as linhas a cada frame
RunService.RenderStepped:Connect(function()
	if not espAtivado then
		for _, dados in pairs(objetosESP) do
			if dados.Line then
				dados.Line.Visible = false
			end
		end
		return
	end

	for jogador, dados in pairs(objetosESP) do
		local char = jogador.Character
		if char and char:FindFirstChild("UpperTorso") then
			local torso = char.UpperTorso
			local pos, vis = Camera:WorldToViewportPoint(torso.Position)
			if vis then
				dados.Line.Visible = true
				dados.Line.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y * 0.85)
				dados.Line.To = Vector2.new(pos.X, pos.Y)
			else
				dados.Line.Visible = false
			end
		elseif dados.Line then
			dados.Line.Visible = false
		end
	end
end)

-- Atualizações ao entrar/sair
Players.PlayerAdded:Connect(function(jogador)
	if espAtivado then criarESP(jogador) end
end)

Players.PlayerRemoving:Connect(function(jogador)
	removerESP(jogador)
end)

-- Toggle ESP integrado na Hub (SEM notificação ao iniciar)
local Toggle2 = Tab2:AddToggle({
	Name = "ESP",
	Description = "LIGAR ESP LINE",
	Default = false,
	Callback = function(Value)
		-- Ignorar primeira execução (inicialização)
		if not espInicializado then
			espInicializado = true
			return
		end
		
		espAtivado = Value
		
		if espAtivado then
			-- Ativar ESP para todos os jogadores
			for _, jogador in pairs(Players:GetPlayers()) do
				criarESP(jogador)
			end
			-- Notificação APENAS quando ativar manualmente
			game.StarterGui:SetCore("SendNotification", {
				Title = "ESP ATIVADO",
				Text = "",
				Duration = 1.
			})
		else
			-- Desativar ESP
			for jogador in pairs(objetosESP) do
				removerESP(jogador)
			end
			-- Notificação APENAS quando desativar manualmente
			game.StarterGui:SetCore("SendNotification", {
				Title = "ESP DESATIVADO",
				Text = "",
				Duration = 1.5
			})
		end
	end
})
