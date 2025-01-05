local Players = game:GetService("Players")

-- Criação da GUI
local function createGui()
    local player = Players.LocalPlayer
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "TeleportGui"
    screenGui.Parent = player:WaitForChild("PlayerGui")

    -- Botão para abrir/fechar a GUI
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0, 150, 0, 40)
    toggleButton.Position = UDim2.new(0, 10, 0, 10)
    toggleButton.Text = "-ABRIR MENU-" -- Texto de abertura
    toggleButton.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
    toggleButton.TextColor3 = Color3.new(1, 1, 1)
    toggleButton.TextScaled = true
    toggleButton.Parent = screenGui

    -- Frame principal da GUI
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0.3, 0, 0.5, 0)
    frame.Position = UDim2.new(0.35, 0, 0.25, 0)
    frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    frame.Visible = false -- Começa oculto
    frame.Parent = screenGui

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0.1, 0)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.Text = "Selecione um jogador para teletransportar"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.SourceSansBold
    title.TextScaled = true
    title.Parent = frame

    local scrollingFrame = Instance.new("ScrollingFrame")
    scrollingFrame.Size = UDim2.new(1, 0, 0.9, -30)
    scrollingFrame.Position = UDim2.new(0, 0, 0.1, 0)
    scrollingFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0) -- Ajuste inicial
    scrollingFrame.ScrollBarThickness = 6
    scrollingFrame.Parent = frame

    -- Função para criar botões
    local function createButton(playerName, yOffset)
        local button = Instance.new("TextButton")
        button.Size = UDim2.new(1, -10, 0, 30)
        button.Position = UDim2.new(0, 5, 0, yOffset)
        button.Text = playerName
        button.TextScaled = true
        button.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
        button.TextColor3 = Color3.new(1, 1, 1)
        button.Font = Enum.Font.SourceSansBold

        -- Teletransporta o jogador ao clicar no botão
        button.MouseButton1Click:Connect(function()
            local targetPlayer = Players:FindFirstChild(playerName)
            if targetPlayer and targetPlayer.Character and player.Character then
                local targetPosition = targetPlayer.Character.PrimaryPart.Position
                player.Character:SetPrimaryPartCFrame(CFrame.new(targetPosition + Vector3.new(0, 5, 0)))
            end
        end)

        button.Parent = scrollingFrame
    end

    -- Atualiza a lista de jogadores
    local function updatePlayerList()
        scrollingFrame:ClearAllChildren() -- Limpa a lista
        local yOffset = 0 -- Controle da posição vertical dos botões

        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player then -- Ignora o jogador local
                createButton(p.Name, yOffset)
                yOffset = yOffset + 35 -- Move o próximo botão para baixo
            end
        end

        -- Ajusta o tamanho da CanvasSize dinamicamente
        scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, yOffset)
    end

    -- Alterna a visibilidade do frame
    local function toggleFrame()
        frame.Visible = not frame.Visible
        toggleButton.Text = frame.Visible and "-FECHAR MENU-" or "-ABRIR MENU-" -- Atualiza o texto
    end

    -- Conecta o botão de abrir/fechar
    toggleButton.MouseButton1Click:Connect(toggleFrame)

    -- Atualiza a lista de jogadores quando alguém entra ou sai do jogo
    Players.PlayerAdded:Connect(updatePlayerList)
    Players.PlayerRemoving:Connect(updatePlayerList)

    -- Inicializa a lista
    updatePlayerList()

    -- Função para mover a GUI
    local dragging = false
    local dragInput, mousePos, framePos

    -- Quando começa a arrastar
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragInput = input
            mousePos = input.Position
            framePos = frame.Position
        end
    end)

    -- Quando o arrasto continua
    frame.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local delta = input.Position - mousePos
            frame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
        end
    end)

    -- Quando o arrasto termina
    frame.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
end

-- Cria a GUI e reconecta após respawn
local player = Players.LocalPlayer
player.CharacterAdded:Connect(function()
    createGui()
end)

-- Cria a GUI inicial
createGui()
