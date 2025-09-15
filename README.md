local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local PathfindingService = game:GetService("PathfindingService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

-- Aqui você deve colocar os nomes reais depois de descobrir com o script de listagem
local NOME_SACO = "NomeDoSacoAqui" -- Exemplo: "TrashBag"
local EVENTO_COLETAR = "NomeDoEventoAqui" -- Exemplo: "CollectTrashEvent"

local coletaEvent = ReplicatedStorage:WaitForChild(EVENTO_COLETAR)

local autoFarmAtivo = false
local distanciaColeta = 5
local tempoEsperaColeta = 3

local function encontrarSacoMaisProximo()
    local maisProximo = nil
    local menorDistancia = math.huge
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name == NOME_SACO then
            local dist = (hrp.Position - obj.Position).Magnitude
            if dist < menorDistancia then
                menorDistancia = dist
                maisProximo = obj
            end
        end
    end
    return maisProximo
end

local function moverPara(destinoPos)
    local path = PathfindingService:CreatePath()
    path:ComputeAsync(hrp.Position, destinoPos)
    if path.Status == Enum.PathStatus.Success then
        for _, waypoint in ipairs(path:GetWaypoints()) do
            if not autoFarmAtivo then return false end
            humanoid:MoveTo(waypoint.Position)
            local reached = humanoid.MoveToFinished:Wait()
            if not reached then return false end
        end
        return true
    else
        warn("Pathfinding falhou:", path.Status.Name)
        return false
    end
end

local function autoFarm()
    while autoFarmAtivo do
        local saco = encontrarSacoMaisProximo()
        if not saco then
            warn("Nenhum saco encontrado.")
            task.wait(3)
        else
            local sucesso = moverPara(saco.Position + Vector3.new(0,3,0))
            if not sucesso then
                task.wait(1)
            else
                while (hrp.Position - saco.Position).Magnitude > distanciaColeta do
                    if not autoFarmAtivo then break end
                    task.wait(0.1)
                end

                coletaEvent:FireServer(saco)
                print("[AutoFarm] Saco coletado!")
                task.wait(tempoEsperaColeta)
            end
        end
    end
end

-- Criar GUI simples para ligar/desligar o Auto Farm
local ScreenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
ScreenGui.Name = "AutoFarmMenu"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 250, 0, 100)
Frame.Position = UDim2.new(0, 20, 0.5, -50)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0

local ToggleButton = Instance.new("TextButton", Frame)
ToggleButton.Size = UDim2.new(0, 230, 0, 50)
ToggleButton.Position = UDim2.new(0, 10, 0, 25)
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
ToggleButton.TextColor3 = Color3.new(1,1,1)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 24
ToggleButton.Text = "Ativar Auto Farm Gari"

ToggleButton.MouseButton1Click:Connect(function()
    autoFarmAtivo = not autoFarmAtivo
    if autoFarmAtivo then
        ToggleButton.Text = "Desativar Auto Farm Gari"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
        spawn(autoFarm)
    else
        ToggleButton.Text = "Ativar Auto Farm Gari"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
    end
end)
