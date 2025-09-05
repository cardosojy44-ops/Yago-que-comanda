--==================================================
-- PAINEL ADM + COMANDOS CHAT
--==================================================
-- LocalScript deve ir em StarterGui
-- Crie um RemoteEvent chamado "AdminEvent" em ReplicatedStorage
--==================================================

if game:GetService("RunService"):IsClient() then
	-------------------------------------------------
	-- [ CLIENTE / INTERFACE ]
	-------------------------------------------------
	local Players = game:GetService("Players")
	local ReplicatedStorage = game:GetService("ReplicatedStorage")
	local player = Players.LocalPlayer
	local event = ReplicatedStorage:WaitForChild("AdminEvent")

	-- Criar ScreenGui
	local gui = Instance.new("ScreenGui")
	gui.Name = "PainelADM"
	gui.Parent = player:WaitForChild("PlayerGui")

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 320, 0, 320)
	frame.Position = UDim2.new(0.3, 0, 0.3, 0)
	frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
	frame.Active = true
	frame.Draggable = true
	frame.Parent = gui

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 12)
	corner.Parent = frame

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

	local minimizar = Instance.new("TextButton")
	minimizar.Size = UDim2.new(0, 30, 0, 30)
	minimizar.Position = UDim2.new(1, -35, 0, 5)
	minimizar.Text = "-"
	minimizar.Font = Enum.Font.GothamBold
	minimizar.TextSize = 22
	minimizar.TextColor3 = Color3.fromRGB(255, 255, 255)
	minimizar.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
	minimizar.Parent = frame

	local minimizado = false
	minimizar.MouseButton1Click:Connect(function()
		if minimizado then
			frame.Size = UDim2.new(0, 320, 0, 320)
			minimizado = false
		else
			frame.Size = UDim2.new(0, 320, 0, 40)
			minimizado = true
		end
	end)

	local dropdown = Instance.new("TextButton")
	dropdown.Size = UDim2.new(0, 280, 0, 30)
	dropdown.Position = UDim2.new(0, 20, 0, 50)
	dropdown.Text = "Selecionar Player"
	dropdown.Font = Enum.Font.Gotham
	dropdown.TextSize = 16
	dropdown.TextColor3 = Color3.fromRGB(255, 255, 255)
	dropdown.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	dropdown.Parent = frame

	local cornerDrop = Instance.new("UICorner")
	cornerDrop.CornerRadius = UDim.new(0, 8)
	cornerDrop.Parent = dropdown

	local listaFrame = Instance.new("Frame")
	listaFrame.Size = UDim2.new(0, 280, 0, 120)
	listaFrame.Position = UDim2.new(0, 20, 0, 85)
	listaFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	listaFrame.Visible = false
	listaFrame.Parent = frame

	local layout = Instance.new("UIListLayout")
	layout.Parent = listaFrame
	layout.Padding = UDim.new(0, 2)

	local escolhido = nil
	local function atualizarLista()
		for _, c in pairs(listaFrame:GetChildren()) do
			if c:IsA("TextButton") then c:Destroy() end
		end
		for _, p in pairs(Players:GetPlayers()) do
			if p ~= player then
				local btn = Instance.new("TextButton")
				btn.Size = UDim2.new(1, 0, 0, 25)
				btn.Text = p.Name
				btn.Font = Enum.Font.Gotham
				btn.TextSize = 14
				btn.TextColor3 = Color3.fromRGB(255, 255, 255)
				btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
				btn.Parent = listaFrame
				btn.MouseButton1Click:Connect(function()
					escolhido = p.Name
					dropdown.Text = "Player: " .. p.Name
					listaFrame.Visible = false
				end)
			end
		end
	end

	dropdown.MouseButton1Click:Connect(function()
		listaFrame.Visible = not listaFrame.Visible
		atualizarLista()
	end)

	local function criarBotao(texto, acao)
		local btn = Instance.new("TextButton")
		btn.Size = UDim2.new(0, 280, 0, 30)
		btn.Text = texto
		btn.Font = Enum.Font.Gotham
		btn.TextSize = 16
		btn.TextColor3 = Color3.fromRGB(255, 255, 255)
		btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
		btn.Parent = frame

		local cornerBtn = Instance.new("UICorner")
		cornerBtn.CornerRadius = UDim.new(0, 8)
		cornerBtn.Parent = btn

		btn.MouseButton1Click:Connect(function()
			if escolhido then
				event:FireServer(acao, escolhido)
			end
		end)
	end

	criarBotao("Kill Player", "Kill")
	criarBotao("Expulsar Player", "Kick")
	criarBotao("Banir Player", "Ban")
	criarBotao("Fly Player", "Fly")
	criarBotao("Jail Player (5s)", "Jail")

	local function adicionarTagADM(char)
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
	end

	player.CharacterAdded:Connect(adicionarTagADM)
	if player.Character then adicionarTagADM(player.Character) end

else
	-------------------------------------------------
	-- [ SERVIDOR / COMANDOS E BAN PERMANENTE ]
	-------------------------------------------------
	local ReplicatedStorage = game:GetService("ReplicatedStorage")
	local Players = game:GetService("Players")
	local DataStoreService = game:GetService("DataStoreService")
	local StarterGui = game:GetService("StarterGui")

	local BanStore = DataStoreService:GetDataStore("BanidosStore")
	local event = ReplicatedStorage:WaitForChild("AdminEvent")

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

	local function salvarBan(player)
		pcall(function() BanStore:SetAsync(player.UserId, true) end)
	end

	local function removerBan(userId)
		pcall(function() BanStore:SetAsync(userId, false) end)
	end

	-- Executar ações
	local function executarComando(acao, alvo)
		if not alvo then return end
		if acao == "Kill" and alvo.Character and alvo.Character:FindFirstChild("Humanoid") then
			alvo.Character.Humanoid.Health = 0
			anunciar("☠️ " .. alvo.Name .. " foi morto pelo ADM Yago!", Color3.fromRGB(255,100,100))

		elseif acao == "Kick" then
			anunciar("🚪 " .. alvo.Name .. " foi EXPULSO pelo ADM Yago!", Color3.fromRGB(255,200,0))
			alvo:Kick("Você foi expulso pelo ADM.")

		elseif acao == "Ban" then
			salvarBan(alvo)
			anunciar("⛔ " .. alvo.Name .. " foi BANIDO pelo ADM Yago!", Color3.fromRGB(255,0,0))
			alvo:Kick("Você foi BANIDO permanentemente pelo ADM.")

		elseif acao == "Fly" and alvo.Character then
			local hrp = alvo.Character:FindFirstChild("HumanoidRootPart")
			if hrp then
				hrp.Anchored = not hrp.Anchored
				anunciar("🕊️ " .. alvo.Name .. " teve Fly " .. (hrp.Anchored and "ATIVADO" or "DESATIVADO") .. " pelo ADM Yago!", Color3.fromRGB(150,200,255))
			end

		elseif acao == "Jail" and alvo.Character then
			local hrp = alvo.Character:FindFirstChild("HumanoidRootPart")
			if hrp then
				hrp.CFrame = CFrame.new(0,5,0)
				hrp.Anchored = true
				anunciar("🔒 " .. alvo.Name .. " foi preso por 5s pelo ADM Yago!", Color3.fromRGB(200,150,255))
				task.delay(5, function()
					if hrp then hrp.Anchored = false end
				end)
			end
		end
	end

	Players.PlayerAdded:Connect(function(player)
		if carregarBan(player) then
			player:Kick("Você está BANIDO permanentemente do servidor.")
		end

		-- Comandos de chat
		player.Chatted:Connect(function(msg)
			if player.Name == "Yago" then
				local args = string.split(msg, " ")
				local comando = string.lower(args[1])
				local alvoNome = args[2]
				local alvo = alvoNome and Players:FindFirstChild(alvoNome)

				if comando == "!ban" and alvoNome then
					if alvo then
						executarComando("Ban", alvo)
					elseif tonumber(alvoNome) then
						local userId = tonumber(alvoNome)
						salvarBan({UserId=userId})
						anunciar("⛔ UserId " .. userId .. " foi BANIDO pelo ADM Yago!", Color3.fromRGB(255,0,0))
					end

				elseif comando == "!unban" and alvoNome then
					if alvo then
						removerBan(alvo.UserId)
						anunciar("✅ " .. alvo.Name .. " foi DESBANIDO pelo ADM Yago!", Color3.fromRGB(0,255,0))
					elseif tonumber(alvoNome) then
						removerBan(tonumber(alvoNome))
						anunciar("✅ UserId " .. alvoNome .. " foi DESBANIDO pelo ADM Yago!", Color3.fromRGB(0,255,0))
					end

				elseif comando == "!kick" and alvo then
					executarComando("Kick", alvo)

				elseif comando == "!kill" and alvo then
					executarComando("Kill", alvo)

				elseif comando == "!fly" and alvo then
					executarComando("Fly", alvo)

				elseif comando == "!jail" and alvo then
					executarComando("Jail", alvo)
				end
			end
		end)
	end)

	-- Painel dispara eventos
	event.OnServerEvent:Connect(function(sender, acao, alvoNome)
		if sender.Name ~= "Yago" then return end
		local alvo = Players:FindFirstChild(alvoNome)
		executarComando(acao, alvo)
	end)
end
