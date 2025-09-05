--// Painel ADM Local - Delta Android
--// Autor: YAGO QUE COMANDA 👑

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PainelADM"
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 430)
Frame.Position = UDim2.new(0.7, 0, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner", Frame)
UICorner.CornerRadius = UDim.new(0, 12)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -10, 0, 40)
Title.Position = UDim2.new(0, 5, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "YAGO QUE COMANDA"
Title.Font = Enum.Font.GothamBold
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Center
Title.Parent = Frame

-- Funções Locais
local player = game.Players.LocalPlayer

-- Tag ADM
local function addTagADM(char)
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

player.CharacterAdded:Connect(addTagADM)
if player.Character then addTagADM(player.Character) end

-- Criar Botões
local function criarBotao(nome, yPos, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 220, 0, 30)
    btn.Position = UDim2.new(0, 15, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.Text = nome
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.Parent = Frame
    local corner = Instance.new("UICorner", btn)
    corner.CornerRadius = UDim.new(0, 8)
    btn.MouseButton1Click:Connect(callback)
end

-- Fly
local flying = false
criarBotao("🕊️ Fly", 50, function()
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if hrp then
        flying = not flying
        if flying then
            hrp.Anchored = true
            hrp.Velocity = Vector3.new(0,0,0)
        else
            hrp.Anchored = false
        end
    end
end)

-- Speed
local fast = false
criarBotao("⚡ Speed", 90, function()
    local hum = player.Character and player.Character:FindFirstChild("Humanoid")
    if hum then
        fast = not fast
        hum.WalkSpeed = fast and 100 or 16
    end
end)

-- GodMode
local god = false
criarBotao("💎 GodMode", 130, function()
    local hum = player.Character and player.Character:FindFirstChild("Humanoid")
    if hum then
        god = not god
        if god then
            hum.Name = "1xGod"
            hum.MaxHealth = math.huge
            hum.Health = math.huge
        else
            hum.Name = "Humanoid"
            hum.MaxHealth = 100
            hum.Health = 100
        end
    end
end)

-- Reset
criarBotao("☠️ Reset", 170, function()
    local hum = player.Character and player.Character:FindFirstChild("Humanoid")
    if hum then hum.Health = 0 end
end)

-- ESP
local espAtivo = false
criarBotao("👀 ESP", 210, function()
    espAtivo = not espAtivo
    for _,plr in pairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
            if espAtivo then
                local bill = Instance.new("BillboardGui")
                bill.Name = "ESP"
                bill.Size = UDim2.new(0,100,0,30)
                bill.Adornee = plr.Character.Head
                bill.AlwaysOnTop = true
                bill.Parent = plr.Character.Head

                local txt = Instance.new("TextLabel")
                txt.Size = UDim2.new(1,0,1,0)
                txt.BackgroundTransparency = 1
                txt.Text = plr.Name
                txt.TextColor3 = Color3.fromRGB(0,255,0)
                txt.Font = Enum.Font.GothamBold
                txt.TextSize = 14
                txt.Parent = bill
            else
                if plr.Character and plr.Character:FindFirstChild("Head") and plr.Character.Head:FindFirstChild("ESP") then
                    plr.Character.Head.ESP:Destroy()
                end
            end
        end
    end
end)

-- Tracers
local tracersAtivo = false
local runService = game:GetService("RunService")
local camera = game.Workspace.CurrentCamera
local tracers = {}

criarBotao("📦 Tracers", 250, function()
    tracersAtivo = not tracersAtivo
    if not tracersAtivo then
        for _,line in pairs(tracers) do line:Remove() end
        tracers = {}
    end
end)

runService.RenderStepped:Connect(function()
    if tracersAtivo then
        for _,line in pairs(tracers) do line:Remove() end
        tracers = {}
        for _,plr in pairs(game.Players:GetPlayers()) do
            if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = plr.Character.HumanoidRootPart
                local vector, onScreen = camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local line = Drawing.new("Line")
                    line.From = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y)
                    line.To = Vector2.new(vector.X, vector.Y)
                    line.Color = Color3.fromRGB(255,0,0)
                    line.Thickness = 2
                    line.Visible = true
                    table.insert(tracers, line)
                end
            end
        end
    end
end)

-- Invisibilidade
local invisivel = false
criarBotao("👻 Invisível", 290, function()
    local char = player.Character
    if not char then return end
    invisivel = not invisivel
    for _,part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            part.Transparency = invisivel and 1 or 0
        end
        if part:IsA("Accessory") and part:FindFirstChild("Handle") then
            part.Handle.Transparency = invisivel and 1 or 0
        end
    end
end)

-- Teleporte
criarBotao("🚀 Teleport", 330, function()
    local teleportBox = Instance.new("TextBox")
    teleportBox.Size = UDim2.new(0, 220, 0, 30)
    teleportBox.Position = UDim2.new(0, 15, 0, 370)
    teleportBox.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    teleportBox.Text = "Digite o nome do player"
    teleportBox.TextColor3 = Color3.fromRGB(255,255,255)
    teleportBox.ClearTextOnFocus = true
    teleportBox.Font = Enum.Font.Gotham
    teleportBox.TextSize = 14
    teleportBox.Parent = Frame
    local corner = Instance.new("UICorner", teleportBox)
    corner.CornerRadius = UDim.new(0, 8)

    teleportBox.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local nome = teleportBox.Text
            local target = game.Players:FindFirstChild(nome)
            if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.CFrame = target.Character.HumanoidRootPart.CFrame + Vector3.new(0,3,0)
                end
            end
        end
    end)
end)
