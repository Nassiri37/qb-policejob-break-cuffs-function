# qb-policejob-break-cuffs-function

# Add to Config.lua
Config.Minigame = "circleminigame" -- "lib.skillcheck" or "ps-ui" or "circleminigame"

# Add to server/main.lua
RegisterNetEvent('notifyCuffBreak', function(playerId)
    TriggerClientEvent('QBCore:Notify', playerId, "Seems this person broke out of their handcuffs.", "error", 3000)
end)

# In client/interactions.lua

# Add
to lines 3 and 4
local cuffedtimes = 0
local maxattempts = 3

# Search for and replace police:client:GetCuffed





RegisterNetEvent('police:client:GetCuffed', function(playerId, isSoftcuff, AttachEntity)
    local lastStand = QBCore.Functions.GetPlayerData().metadata["inlaststand"]
    local deadBozo = QBCore.Functions.GetPlayerData().metadata["isdead"]
    local ped = PlayerPedId()

    local function handleCuffNotificationAndType(type, message)
        isHandcuffed = true
        TriggerServerEvent("police:server:SetHandcuffStatus", type)
        GetCuffedAnimation(playerId)
        QBCore.Functions.Notify(message, 'primary')
        return type
    end

    if lastStand or deadBozo then
        QBCore.Functions.Notify("You cannot attempt to break free in your current state.")
        cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                    handleCuffNotificationAndType(16, Lang:t("info.cuff"))
        return 
    end

