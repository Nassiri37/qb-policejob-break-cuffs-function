# qb-policejob-break-cuffs-function
```
Currently supports ox_lib skillcheck / ps-ui / circleminigame by trclassic92

Gives Cop/Player notifications when cuffs are broken

Players in laststand/dead dont have the option to break cuffs

After player beats Config.MaxCuffAttempts the next time they are cuffed they wont be shown a minigame and will be cuffed

Adds handcuff prop onto players
```

# Add to Config.lua
``` 
Config.Minigame = "circleminigame" -- "lib.skillcheck" or "ps-ui" or "circleminigame"
Config.MaxCuffAttempts = 3 -- Or however many you want

-- ox_lib skillcheck
Config.Keys = {'W','A','S','D'}
Config.Difficulty = {'easy','easy','easy'} -- easy, medium, hard // can add as many more as you want or less

-- ps-ui
Config.Circles = 3 -- How many circles
Config.ms = 10 -- How fast or slow it passes

-- circleminigame by trclassic92
Config.Time = math.random(6,10) -- // can be solid number or math random ex. Config.Time = 5
Config.TRCircles = math.random(2, 5) -- // can be solid number or math random ex. Config.TRCircles = 5 
```


# Add to server/main.lua
```
RegisterNetEvent('notifyCuffBreak', function(playerId)
    TriggerClientEvent('QBCore:Notify', playerId, "Seems this person broke out of their handcuffs.", "error", 3000)
end)
```

# In client/interactions.lua

**Add
to lines 3 and 4**
```
local cuffedtimes = 0
local maxattempts = Config.MaxCuffAttempts
```

**Search for police:client:GetCuffed and place above**
```
function AttachEntityToPlayer(prop)
    if prop then
        QBCore.Functions.LoadModel(prop)
        local ped = PlayerPedId()
        local pos = GetEntityCoords(ped)
        local ent = CreateObjectNoOffset(prop, pos.x, pos.y, pos.z, 1, 1, 0)
        AttachEntityToEntity(ent, ped, GetPedBoneIndex(ped, 60309), -0.055, 0.06, 0.04, 265.0, 155.0, 80.0, true, false, false, false, 0, true)
        return ent
    end
    return nil
end
```

**Search for and replace police:client:GetCuffed**

```
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
            local keys = Config.Keys
            local difficulty = Config.Difficulty
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
                    local cuffProp = AttachEntityToPlayer("p_cs_cuffs_02_s")
                    ClearPedTasksImmediately(ped)
                    isHandcuffed = true
                    TriggerServerEvent("police:server:SetHandcuffStatus", true)
                    cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                            handleCuffNotificationAndType(16, Lang:t("info.cuff"))
                end
                    if GetSelectedPedWeapon(ped) ~= WEAPON_UNARMED then
                    SetCurrentPedWeapon(ped, WEAPON_UNARMED, true)
                end
               else
                   QBCore.Functions.Notify("You cannot try to break free anymore.")
                    cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                            handleCuffNotificationAndType(16, Lang:t("info.cuff"))
            end
        elseif Config.Minigame == "ps-ui" then
            local circles = Config.Circles
            local ms = Config.ms        
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
                        local cuffProp = AttachEntityToPlayer("p_cs_cuffs_02_s")
                        isHandcuffed = true
                        TriggerServerEvent("police:server:SetHandcuffStatus", true)
                        cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                                handleCuffNotificationAndType(16, Lang:t("info.cuff"))
                    end
                end, circles, ms)
            else
                QBCore.Functions.Notify("You cannot try to break free anymore.")
                cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                            handleCuffNotificationAndType(16, Lang:t("info.cuff"))
            end
        elseif Config.Minigame == "circleminigame" then
            local time = Config.Time
            local trcircles = Config.TRCircles
            cuffedtimes = cuffedtimes + 1
            if cuffedtimes <= maxattempts then
                local success = exports['CircleMinigame']:StartLockPickCircle(trcircles, time) 
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
                    local cuffProp = AttachEntityToPlayer("p_cs_cuffs_02_s")
                    isHandcuffed = true
                    TriggerServerEvent("police:server:SetHandcuffStatus", true)
                    cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                            handleCuffNotificationAndType(16, Lang:t("info.cuff"))
                end
            else
                QBCore.Functions.Notify("You cannot try to break free anymore.")
                cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                        handleCuffNotificationAndType(16, Lang:t("info.cuff"))
            end
        else
            QBCore.Functions.Notify("You cannot try to break free anymore.")
            cuffType = isSoftcuff and handleCuffNotificationAndType(49, Lang:t("info.cuffed_walk")) or
                                        handleCuffNotificationAndType(16, Lang:t("info.cuff"))
        end
    else
        isHandcuffed = false
        isEscorted = false
        DetachEntity(ent, 0, 0)
        DeleteEntity(ent)

        TriggerEvent('hospital:client:isEscorted', false)
        DetachEntity(ped, true, false)
        TriggerServerEvent("police:server:SetHandcuffStatus", false)
        ClearPedTasksImmediately(ped)
        TriggerServerEvent("InteractSound_SV:PlayOnSource", "Uncuff", 0.2)
        cuffedtimes = 0
        QBCore.Functions.Notify(Lang:t("success.uncuffed"), "success")
    end
end)
```
