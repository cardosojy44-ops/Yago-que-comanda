--==================================================
-- SCRIPT ADM COMPLETO - YAGO QUE COMANDA
-- Coloque em ServerScriptService
--==================================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")
local DataStoreService = game:GetService("DataStoreService")

-- RemoteEvent
local event = Instance.new("RemoteEvent")
event.Name = "AdminEvent"
event.Parent = ReplicatedStorage

-- DataStore de banidos
local BanStore = DataStoreService:GetDataStore("BanidosStore")

-------------------------------------------------
-- Funções auxiliares
-------------------------------------------------
local function anunciar(msg, cor)
	StarterGui:SetCore("ChatMakeSystemMessage", {
		Text = msg;
		Color = cor or Color3.fromRGB(255, 200, 0);
		Font = Enum.Font.GothamBold;
		TextSize = 18;
	})
end

local function carregarBan(player)
	local ok, result = pcall(function()
		return BanStore:GetAsync(player.UserId)
	end)
	return ok and result == true
end

local function salvarBan(userId)
	pcall(function() BanStore:SetAsync(userId, true) end)
end

local function removerBan(userId)
	pcall(function() BanStore:SetAsync(userId, false) end)
end

-------------------------------------------------
-- Ações de ADM
-------------------------------------------------
local function executarComando(acao, alvo, sender)
	if not alvo then return end
	if acao == "Kill" and alvo.Character and alvo.Character:FindFirstChild("Humanoid") then
		alvo.Character.Humanoid.Health = 0
		anunciar("☠️ " .. alvo.Name .. " foi morto pelo ADM " .. sender.Name, Color3.fromRGB(255,100,100))

	elseif acao == "Kick" then
		anunciar("🚪 " .. alvo.Name .. " foi EXPULSO pelo ADM " .. sender.Name, Color3.fromRGB(255,200,0))
		alvo:Kick("Você foi expulso pelo ADM.")

	elseif acao == "Ban" then
		salvarBan(alvo.UserId)
		anunciar("⛔ " .. alvo.Name .. " foi BANIDO pelo ADM " .. sender.Name, Color3.fromRGB(255,0,0))
		alvo:Kick("Você foi BANIDO permanentemente pelo ADM.")

	elseif acao == "Fly" and alvo.Character then
		local hrp = alvo.Character:FindFirstChild("HumanoidRootPart")
		if hrp then
			hrp.Anchored = not hrp.Anchored
			anunciar("🕊️ " .. alvo.Name .. " teve Fly " .. (hrp.Anchored and "ATIVADO" or "DESATIVADO") .. " pelo ADM " .. sender.Name, Color3.fromRGB(150,200,255))
		end

	elseif acao == "Jail" and alvo.Character then
		local hrp = alvo.Character:FindFirstChild("HumanoidRootPart")
		if hrp then
			hrp.CFrame = CFrame.new(0,5,0)
			hrp.Anchored = true
			anunciar("🔒 " .. alvo.Name .. " foi preso por 5s pelo ADM " .. sender.Name, Color3.fromRGB(200,150,255))
			task.delay(5, function()
				if hrp then hrp.Anchored = false end
			end)
		end
	end
end

-------------------------------------------------
-- Chat Commands
-------------------------------------------------
Players.PlayerAdded:Connect(function(player)
	-- Checa banimento ao entrar
	if carregarBan(player) then
		player:Kick("Você está BANIDO permanentemente do servidor.")
		return
	end

	-- Colocar TAG ADM no Yago
	if player.Name == "Yago" then
		player.CharacterAdded:Connect(function(char)
			if char:FindFirstChild("Head") then
				local billboard = Instance.new("BillboardGui")
				billboard.Size = UDim2.new(0, 100, 0, 30)
				billboard.Adornee = char.Head
				billboard.AlwaysOnTop = true
				billboard.Parent = char.Head

				local texto = Instance.new("TextLabel")
				texto.Size = UDim2.new(1, 0, 1, 0)
				texto.BackgroundTransparency = 1
				texto.Text = "ADM"
				texto.Font = Enum.Font.GothamBold
				texto.TextColor3 = Color3.fromRGB(255, 50, 50)
				texto.TextStrokeTransparency = 0
				texto.TextSize = 18
				texto.Parent = billboard
			end
		end)
	end

	-- Sistema de comandos pelo chat
	player.Chatted:Connect(function(msg)
		if player.Name ~= "Yago" then return end
		local args = string.split(msg, " ")
		local comando = string.lower(args[1])
		local alvoNome = args[2]
		local alvo = alvoNome and Players:FindFirstChild(alvoNome)

		if comando == "!ban" and alvo then
			executarComando("Ban", alvo, player)

		elseif comando == "!unban" and alvoNome then
			if alvo then
				removerBan(alvo.UserId)
				anunciar("✅ " .. alvo.Name .. " foi DESBANIDO pelo ADM " .. player.Name, Color3.fromRGB(0,255,0))
			elseif tonumber(alvoNome) then
				removerBan(tonumber(alvoNome))
				anunciar("✅ UserId " .. alvoNome .. " foi DESBANIDO pelo ADM " .. player.Name, Color3.fromRGB(0,255,0))
			end

		elseif comando == "!kick" and alvo then
			executarComando("Kick", alvo, player)

		elseif comando == "!kill" and alvo then
			executarComando("Kill", alvo, player)

		elseif comando == "!fly" and alvo then
			executarComando("Fly", alvo, player)

		elseif comando == "!jail" and alvo then
			executarComando("Jail", alvo, player)
		end
	end)
end)

-------------------------------------------------
-- Painel GUI (criado para Yago)
-------------------------------------------------
event.OnServerEvent:Connect(function(sender, acao, alvoNome)
	if sender.Name ~= "Yago" then return end
	local alvo = Players:FindFirstChild(alvoNome)
	executarComando(acao, alvo, sender)
end)

-- Painel GUI é enviado para o PlayerGui de Yago
Players.PlayerAdded:Connect(function(player)
	if player.Name == "Yago" then
		local guiSource = script:FindFirstChild("PainelGui")
		if not guiSource then
			-- Cria GUI dinamicamente
			local gui = Instance.new("ScreenGui")
			gui.Name = "PainelADM"

			local frame = Instance.new("Frame")
			frame.Size = UDim2.new(0, 320, 0, 320)
			frame.Position = UDim2.new(0.3, 0, 0.3, 0)
			frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
			frame.Active = true
			frame.Draggable = true
			frame.Parent = gui

			local titulo = Instance.new("TextLabel")
			titulo.Size = UDim2.new(1, -40, 0, 40)
			titulo.Position = UDim2.new(0, 10, 0, 0)
			titulo.BackgroundTransparency = 1
			titulo.Text = "YAGO QUE COMANDA"
			titulo.Font = Enum.Font.GothamBold
			titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
			titulo.TextSize = 18
			titulo.TextXAlignment = Enum.TextXAlignment.Left
			titulo.Parent = frame

			gui.Parent = player:WaitForChild("PlayerGui")
		end
	end
end)

