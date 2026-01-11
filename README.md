local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local toggleEvent = ReplicatedStorage:WaitForChild("AutoFarmToggle")

local activePlayers = {}

-- CONFIGURAÇÕES
local QUEST_FOLDER = workspace:WaitForChild("Quests") -- NPCs de missão
local ENEMY_FOLDER = workspace:WaitForChild("Enemies") -- Mobs
local ATTACK_DAMAGE = 50
local ATTACK_INTERVAL = 0.5

-- Liga / desliga
toggleEvent.OnServerEvent:Connect(function(player, state)
	activePlayers[player] = state
end)

-- Função exemplo de level
local function giveLevel(player)
	local leaderstats = player:FindFirstChild("leaderstats")
	if leaderstats and leaderstats:FindFirstChild("Level") then
		leaderstats.Level.Value += 1
	end
end

-- Função principal
task.spawn(function()
	while true do
		task.wait(ATTACK_INTERVAL)

		for player, isActive in pairs(activePlayers) do
			if isActive and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
				
				local character = player.Character
				local hrp = character.HumanoidRootPart

				-- 1. Pega missão (simples)
				local questNpc = QUEST_FOLDER:FindFirstChildWhichIsA("Model")
				if questNpc and questNpc:FindFirstChild("QuestRemote") then
					questNpc.QuestRemote:Fire(player)
				end

				-- 2. Procura inimigo
				local enemy = ENEMY_FOLDER:FindFirstChildWhichIsA("Model")
				if enemy and enemy:FindFirstChild("Humanoid") and enemy:FindFirstChild("HumanoidRootPart") then

					-- 3. Teleporta até o inimigo
					hrp.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)

					-- 4. Ataca automaticamente
					enemy.Humanoid:TakeDamage(ATTACK_DAMAGE)

					-- 5. Se matou, ganha level
					if enemy.Humanoid.Health <= 0 then
						giveLevel(player)
					end
				end
			end
		end
	end
end)

-- Limpeza
Players.PlayerRemoving:Connect(function(player)
	activePlayers[player] = nil
end)
