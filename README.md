-- Configuração
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Tabela para armazenar ESPs ativos
local ESPs = {}

-- Função para criar o ESP para um jogador
local function criarESP(player)
    if ESPs[player] then return end

    local esp = Drawing.new("Text")
    esp.Size = 13
    esp.Center = true
    esp.Outline = true
    esp.Visible = false
    esp.Font = 2 -- UI font
    ESPs[player] = esp
end

-- Função para remover ESP
local function removerESP(player)
    if ESPs[player] then
        ESPs[player]:Remove()
        ESPs[player] = nil
    end
end

-- Monitorar jogadores
Players.PlayerAdded:Connect(criarESP)
Players.PlayerRemoving:Connect(removerESP)
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        criarESP(player)
    end
end

-- Atualização contínua do ESP
RunService.RenderStepped:Connect(function()
    for player, esp in pairs(ESPs) do
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") then
            local hrp = char.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

            if onScreen then
                local distancia = (hrp.Position - Camera.CFrame.Position).Magnitude
                local vida = math.floor(char.Humanoid.Health)
                local teamColor = player.Team and player.Team.TeamColor.Color or Color3.new(1,1,1)

                esp.Position = Vector2.new(screenPos.X, screenPos.Y - 15)
                esp.Text = string.format("[%s] %dm | %d HP", player.Name, math.floor(distancia), vida)
                esp.Color = teamColor
                esp.Visible = true
            else
                esp.Visible = false
            end
        else
            esp.Visible = false
        end
    end
end)
