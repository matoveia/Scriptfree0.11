local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()
local RayParams = RaycastParams.new()

-- Nome exato da arma
local ArmaPermitida = "Arisaka 7.7mm"
local AimbotEnabled = false
local InimigosVermelhos = {} -- Lista de inimigos vermelhos
local DistanceLimit = 1000 -- Distância máxima para raycast

-- Configurações para raycast (ignorar obstáculos)
RayParams.IgnoreWater = true
RayParams.FilterType = Enum.RaycastFilterType.Blacklist
RayParams.FilterDescendantsInstances = {LocalPlayer.Character}  -- Ignorar o próprio personagem

-- Função para verificar se está segurando a arma certa
local function TemArmaCerta()
    local Character = LocalPlayer.Character
    if Character and Character:FindFirstChildOfClass("Tool") then
        local Arma = Character:FindFirstChildOfClass("Tool")
        return Arma.Name == ArmaPermitida
    end
    return false
end

-- Atualiza a lista de inimigos vermelhos
local function AtualizarInimigos()
    InimigosVermelhos = {}
    for _, Player in pairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") then
            if Player.TeamColor == BrickColor.new("Bright red") then
                table.insert(InimigosVermelhos, Player.Character.Head)
            end
        end
    end
end

-- Função para mirar no inimigo mais próximo (com base no mouse)
local function MirarNosInimigos()
    local Target = nil
    local ShortestDistance = math.huge
    
    -- Atualizar a lista de inimigos vermelhos
    AtualizarInimigos()
    
    for _, Inimigo in pairs(InimigosVermelhos) do
        local EnemyPos, OnScreen = Camera:WorldToViewportPoint(Inimigo.Position)
        if OnScreen then
            local DistanceToCenter = (Vector2.new(EnemyPos.X, EnemyPos.Y) - Mouse.Position).Magnitude
            if DistanceToCenter < ShortestDistance then
                ShortestDistance = DistanceToCenter
                Target = Inimigo
            end
        end
    end
    return Target
end

-- Função para disparar as balas através das paredes
local function ShootRay(Target)
    if TemArmaCerta() and Target then
        local StartPosition = Camera.CFrame.Position
        local Direction = (Target.Position - StartPosition).unit * DistanceLimit
        
        -- Raycast para detectar o alvo
        local RaycastResult = workspace:Raycast(StartPosition, Direction, RayParams)

        -- Verifica se o raio atingiu um inimigo
        if RaycastResult then
            local HitPart = RaycastResult.Instance
            local HitPosition = RaycastResult.Position

            -- Verifica se atingiu um inimigo
            if HitPart and HitPart.Parent and HitPart.Parent:FindFirstChild("Humanoid") then
                local TargetPlayer = Players:GetPlayerFromCharacter(HitPart.Parent)
                if TargetPlayer and TargetPlayer.TeamColor == BrickColor.new("Bright red") then
                    -- Atingiu um inimigo
                    print("Inimigo atingido!")
                end
            end
        end
    end
end

-- Função principal do Aimbot
local function Aimbot()
    while true do
        if AimbotEnabled and TemArmaCerta() then
            local Target = MirarNosInimigos()
            if Target then
                -- Dispara para o inimigo que a mira está grudando
                ShootRay(Target)
            end
        end
        task.wait(0.01)
    end
end

-- Botão para ativar/desativar o Aimbot
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local ToggleButton = Instance.new("TextButton", ScreenGui)

ToggleButton.Size = UDim2.new(0, 200, 0, 50)
ToggleButton.Position = UDim2.new(0.1, 0, 0.1, 0)
ToggleButton.Text = "Aimbot OFF"
ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.BackgroundTransparency = 0.2  

ToggleButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    if AimbotEnabled then
        ToggleButton.Text = "Aimbot ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    else
        ToggleButton.Text = "Aimbot OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

-- Iniciar Aimbot
task.spawn(Aimbot)
