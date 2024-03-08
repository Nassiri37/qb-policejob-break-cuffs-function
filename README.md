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

**Search for and replace police:client:GetCuffed**

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

    if not isHandcuffed then
        if Config.Minigame == "lib.skillcheck" then
            local keys = {'W','A','S','D'}
            local difficulty = {'easy','easy','easy'}
            cuffedtimes = cuffedtimes + 1
            if cuffedtimes <= maxattempts then
                local success = lib.skillCheck(difficulty, keys)
                if success then
                    TriggerServerEvent("police:server:SetHandcuffStatus", false)
                    QBCore.Functions.Notify("You broke free, now run!")
                    TriggerServerEvent('notifyCuffBreak', playerId)
                    isHandcuffed = false
                    isEscorted = false
                    TriggerServerEvent("police:server:SetHandcuffStatus", false)
                    TriggerServerEvent("InteractSound_SV:PlayOnSource", "Uncuff", 0.2)
                    ClearPedTasksImmediately(ped)
                else
                    QBCore.Functions.Notify("You failed to break free!")
                    isHandcuffed = true
                    TriggerServerEvent("police:server:SetHandcuffStatus", true)
                    cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                            handleCuffNotificationAndType(16, Lang:t("info.cuff"))
                end
            end
        elseif Config.Minigame == "ps-ui" then
            local circles = 3
            local ms = 10        
            cuffedtimes = cuffedtimes + 1
            if cuffedtimes <= maxattempts then
                exports['ps-ui']:Circle(function(success)   
                    if success then
                        TriggerServerEvent("police:server:SetHandcuffStatus", false)
                        QBCore.Functions.Notify("You broke free, now run!")
                        TriggerServerEvent('notifyCuffBreak', playerId)
                        isHandcuffed = false
                        isEscorted = false
                        TriggerServerEvent("police:server:SetHandcuffStatus", false)
                        TriggerServerEvent("InteractSound_SV:PlayOnSource", "Uncuff", 0.2)
                        ClearPedTasksImmediately(ped)
                    else
                        QBCore.Functions.Notify("You failed to break free!")
                        isHandcuffed = true
                        TriggerServerEvent("police:server:SetHandcuffStatus", true)
                        cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                                handleCuffNotificationAndType(16, Lang:t("info.cuff"))
                    end
                end, circles, ms)
            end
        elseif Config.Minigame == "circleminigame" then
            local time = math.random(6, 10)
            local circles = math.random(2, 5)
            cuffedtimes = cuffedtimes + 1
            if cuffedtimes <= maxattempts then
                exports['CircleMinigame']:StartLockPickCircle(circles, seconds, success)     
                    if success then
                        TriggerServerEvent("police:server:SetHandcuffStatus", false)
                        QBCore.Functions.Notify("You broke free, now run!")
                        TriggerServerEvent('notifyCuffBreak', playerId)
                        isHandcuffed = false
                        isEscorted = false
                        TriggerServerEvent("police:server:SetHandcuffStatus", false)
                        TriggerServerEvent("InteractSound_SV:PlayOnSource", "Uncuff", 0.2)
                        ClearPedTasksImmediately(ped)
                    else
                        QBCore.Functions.Notify("You failed to break free!")
                        isHandcuffed = true
                        TriggerServerEvent("police:server:SetHandcuffStatus", true)
                        cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                                handleCuffNotificationAndType(16, Lang:t("info.cuff"))
                    end
            
            end
        else
            QBCore.Functions.Notify("You cannot try to break free anymore.")
            cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                        handleCuffNotificationAndType(16, Lang:t("info.cuff"))
        end
    else
        isHandcuffed = false
        isEscorted = false
        TriggerEvent('hospital:client:isEscorted', false)
        DetachEntity(ped, true, false)
        TriggerServerEvent("police:server:SetHandcuffStatus", false)
        ClearPedTasksImmediately(ped)
        TriggerServerEvent("InteractSound_SV:PlayOnSource", "Uncuff", 0.2)
        cuffedtimes = 0
        QBCore.Functions.Notify(Lang:t("success.uncuffed"), "success")
    end
end)
