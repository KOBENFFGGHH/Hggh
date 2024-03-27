do
    CurrentAllMob = {}
    recentlySpawn = 0
    StoredSuccessFully = 0
    canHits = {}
    RecentCollected = 0
    FruitInServer = {}
    RecentlyLocationSet = 0
    lastequip = tick()
end
FirstSea = game.PlaceId == 2753915549
SecondSea = game.PlaceId == 4442272183
ThirdSea = game.PlaceId == 7449423635
SeaIndex = ThirdSea and 3 or SecondSea and 2 or FirstSea and 1 or Client:Kick("Didn't update this Sea")

do
    UpdateQuestData = function(LEVEL,PreviousQuest)
        task.desynchronize()
        local Lvl = math.clamp(LEVEL or Client.Data.Level.Value,MinLevelOfSea,MaxLevelOfSea)
        local less = 1/0
        local AllNearestQuests = {}
        BOSSNAME = nil
        for i,_v in pairs(Quest) do
            for loop = 1,#_v do
                local v = _v[loop]
                if Lvl >= v.LevelReq and not v.MeetsRequirements then
                    if Lvl - v.LevelReq < less then
                        if v.Task[table.foreach(v.Task,tostring)] ~= 1 then
                            QUESTNAME = tostring(i)
                            QUESTNUMBER = tonumber(loop)
                            MONNAME = (table.foreach(v.Task,tostring))
                            QUESTLEVEL = v.LevelReq
                            less = Lvl - v.LevelReq
                            table.insert(AllNearestQuests,1,{
                                name = tostring(i),
                                num = tonumber(loop),
                                mon = (table.foreach(v.Task,tostring)),
                                lvl = v.LevelReq,
                            })
                        end
                    end
                end
            end
        end
        if Lvl < 5000 then
            for i,v in pairs(Quest[QUESTNAME]) do
                if Lvl >= v.LevelReq and not v.MeetsRequirements and v.Task[table.foreach(v.Task,tostring)] == 1 then
                    BOSSQUESTNAME = QUESTNAME
                    BOSSQUESTNUMBER = tonumber(i)
                    BOSSNAME = (table.foreach(v.Task,tostring))
                    BOSSLEVEL = v.LevelReq
                end
            end
        end
        for I,Data in pairs(Guide.Data.NPCList) do
            for Index,Level in pairs(Data.Levels) do
                if Level == QUESTLEVEL then
                    QUESTMOB = Data.NPCName
                    QUESTPOS = CFrame.new(Data.Position)
                end
                if Level == BOSSLEVEL then
                    BOSSMOB = Data.NPCName
                    BOSSPOS = CFrame.new(Data.Position)
                end
                if AllNearestQuests[2] and Level == AllNearestQuests[2].lvl then
                    AllNearestQuests[2].Pos = CFrame.new(Data.Position)
                    AllNearestQuests[2].QuestNPC = Data.NPCName
                end
            end
        end
        if PreviousQuest then
            if AllNearestQuests[2] and QUESTMOB == AllNearestQuests[2].QuestNPC then
                QUESTNAME = AllNearestQuests[2].name
                QUESTNUMBER = AllNearestQuests[2].num
                MONNAME = AllNearestQuests[2].mon
                QUESTLEVEL = AllNearestQuests[2].lvl
                QUESTMOB = AllNearestQuests[2].QuestNPC
                QUESTPOS = AllNearestQuests[2].Pos
            end
        end
        if ("Shanda"):find(MONNAME) or MONNAME == "Wysper" then
            QUESTPOS = CFrame.new(-7860, 5545, -380)
        end
        if ("God's Guard"):find(MONNAME) then
            QUESTPOS = CFrame.new(-4723, 845, -1951)
        end
        if MONNAME:sub(1,#MONNAME) == ("Bandit") then
            QUESTPOS = CFrame.new(1060, 16, 1549)
        end
        if MONNAME:sub(1,#MONNAME) == ("Zombie") then
            QUESTPOS = CFrame.new(-5494, 48, -794)
        end
        task.synchronize()
        return AllNearestQuests
    end
    GetMobName = function()
        if game.Players.LocalPlayer.PlayerGui.Main.Quest.Visible then
            local Text = game.Players.LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text:split(" ")
            local RealText = ""
            local List = #Text
            local Is = true
            if Text[List]:find("/1)") then
                Is = false
            end
            table.remove(Text,1)
            if tonumber(Text[1]) then
                table.remove(Text,1)
            end
            table.remove(Text,#Text)
            for i=1,#Text do local v = Text[i]
                RealText = RealText..v
                if #Text ~= i then
                    RealText = RealText.." "
                end
            end
            if Is then
                RealText = RealText:sub(1,#RealText-1)
            end
            return RealText
        end
    end
    _hasTag = function(TAG,OBJ)
        return CollectionService:HasTag(OBJ or game.Players.LocalPlayer.Character,TAG)
    end
    isnetworkowner = isnetworkowner or function(part)
        local Client = game.Players.LocalPlayer
        if typeof(part) == "Instance" and part:IsA("BasePart") then
            local Distance = math.clamp(Client.SimulationRadius,0,1250)
            local MyDist = Client:DistanceFromCharacter(part.Position)
            if MyDist < Distance then
                for i,v in pairs(game.Players:GetPlayers()) do
                    if v:DistanceFromCharacter(part.Position) < MyDist and v ~= Client then
                        return false
                    end
                end
                return true
            end
        end
    end
    InArea = function(Pos,Location)
        task.desynchronize()
        local nearest,scale = nil,0
        if Location then
            if dist(Pos,Location.Position,true) <= (Location.Mesh.Scale.X/2)+500 then
                return Location
            end
        end
        for i,v in pairs(Locations:GetChildren()) do
            if dist(Pos,v.Position,true) <= (v.Mesh.Scale.X/2)+500 then
                if scale < v.Mesh.Scale.X then
                    scale = v.Mesh.Scale.X
                    nearest = v
                end
            end
        end
        task.synchronize()
        return nearest
    end
    dist = function(a,b,noHeight)
        if not b then
            b = Root.Position
        end
        return (Vector3.new(a.X,not noHeight and a.Y,a.Z) - Vector3.new(b.X,not noHeight and b.Y,b.Z)).magnitude
    end
    _hasNonWeapon = function()
        for i,v in pairs(Backpack:GetChildren()) do
            if v:IsA("Tool") and v.ToolTip == "" and not v:FindFirstChildOfClass("RemoteFunction") then
                return true
            end
        end
    end
    _hasItem = function(name)
        if Client.Backpack:FindFirstChild(name) then return true end
        if Client.Character:FindFirstChild(name) then return true end
        for i,v in pairs(_C("getInventoryWeapons") or {}) do
            if v.Name == name then return v end
        end
    end
    _hasFruit = function()
        local Backpack = Client.Backpack:GetChildren()
        for i=1,#Backpack do local v = Backpack[i]
            if v.Name:lower():find("fruit") then
                return true
            end
        end
        local Character = Client.Character:GetChildren()
        for i=1,#Character do local v = Character[i]
            if v:IsA("Tool") and v.Name:lower():find("fruit") then
                return true
            end
        end
    end
    CollectFruit = function()
        local CollectValue = Priority:get("CollectFruit")
        if not CollectValue:check() then return end
        local Char = Client.Character
        if not Client.Team then _C("SetTeam",Client.Team.Name) end
        if not Char then return end
        if not Char:FindFirstChild("Humanoid") then return end
        if Char:FindFirstChild("Humanoid").Health <= 0 then return end
        if not Root then return end
        if #FruitInServer <= 0 then return end
        -- LeaveSpecificPlace("Cursed Ship")
        local CollectValue = Priority:get("CollectFruit")
        local function Collect(v,CanTween)
            if v.Name == "Fruit " then
                CollectValue:set()
                if Char:FindFirstChild("Humanoid").Health <= 0 or not v:FindFirstChild("Handle") then return end
                repeat
                    CollectValue:set()
                    Root.CFrame = CFrame.new(v:WaitForChild("Handle").Position + Vector3.new(math.random(-3,3),math.random(-5,5),math.random(-3,3)))
                    firetouchinterest(v.Handle,Root,0)
                    task.wait()
                    firetouchinterest(v.Handle,Root,1)
                until v.Parent ~= workspace
                CollectValue:clear()
            end
            if v:IsA("Tool") and v:FindFirstChild("Handle") then
                if dist(v.Handle.Position) <= 5000 then
                    if tick() - RecentCollected < 15 then return end
                    RecentCollected = tick()
                    firetouchinterest(v.Handle,Char.HumanoidRootPart,0)
                    firetouchinterest(v.Handle,Char.HumanoidRootPart,1)
                end
            end
        end
        for i,v in pairs(FruitInServer) do
            if v then
                Collect(v)
                if v.Name == "Fruit " or v:IsA("Tool") then
                    break
                end
            end
        end
        CollectValue:clear()
    end
    FullyBringFruit = function(hop)
        repeat wait() until game.Players.LocalPlayer.Character
        repeat wait() until not CollectFruit()
        wait()
        StoreFruit()
        wait(5)
        if hop then
            Serverhop("Auto Collect Fruit")
        end
    end
    IsCombat = function()
        return Client.PlayerGui.Main.InCombat.Visible
    end
    _hasChip = function()
        local Backpack = Client.Backpack:GetChildren()
        for i=1,#Backpack do local v = Backpack[i]
            if v.Name:lower():find("microchip") then
                return true
            end
        end
        local Character = Client.Character:GetChildren()
        for i=1,#Character do local v = Character[i]
            if v:IsA("Tool") and v.Name:lower():find("microchip") then
                return true
            end
        end
    end
    getFruitDrop = function()
        local fruits = {}
        for i,v in pairs(workspace:GetChildren()) do
            if v:IsA("Tool") or v.Name:find("Fruit") then
                fruits[#fruits+1] = v
            end
        end
        return fruits
    end
    function toServerFruit(Text)
        local Slice = Text:split(":")
        return (Slice[1].."-"..Slice[1]..((Slice[2] and ":"..Slice[2]) or "")):gsub(" Fruit","")
    end
    StoreFruit = function()
        if not DoNotStore then
            local Stored = 0
            local Pattern = "<Color=Green>Collected<Color=/> and <Color=Yellow>stored<Color=/> <Color=Blue>%s<Color=/>"
            local MyFruit = _C("getInventoryFruits")
            for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
                if v.Name:find("Fruit") and not table.find(CollectedFruit,v.Name) then
                    local Name = toServerFruit(v.Name)
                    Noti.new(Pattern:format(v.Name)):Display()
                    _C("StoreFruit",Name,v)
                    Stored = Stored + 1
                    table.insert(CollectedFruit,v.Name)
                end
            end
            for i,v in pairs(game.Players.LocalPlayer.Character:GetChildren()) do
                if v.Name:find("Fruit") and not table.find(CollectedFruit,v.Name) then
                    local Name = toServerFruit(v.Name)
                    Noti.new(Pattern:format(v.Name)):Display()
                    _C("StoreFruit",Name,v)
                    Stored = Stored + 1
                    table.insert(CollectedFruit,v.Name)
                end
            end
            StoredSuccessFully = tick()
        end
    end
    RaidInfo = function()
        local Info = {}
        local Locations = workspace._WorldOrigin.Locations
        local Client = game.Players.LocalPlayer
        local Character = Client.Character
        for i,v in pairs(Locations:GetChildren()) do
            if v.Name:sub(1,6) == "Island" then
                local Distance = (Vector3.new(v.Position.X,0,v.Position.Z) - Vector3.new(Char.HumanoidRootPart.Position.X,0,Char.HumanoidRootPart.Position.Z)).magnitude
                if Distance <= 2000 then
                    Info.ID = v:FindFirstChildOfClass("Folder").Name
                    Info._ID = v:FindFirstChildOfClass("Folder")
                    Info.LastestIsland = v
                    Info.Number = 1
                    break
                end
            end
        end
        function Info.new(INDEX)
            if not Info.ID then return end
            for Do=1,5,1 do
                for i,v in pairs(Locations:GetChildren()) do
                    if v.Name:find(("Island %s"):format(Do)) then
                        if v:FindFirstChild(Info.ID) then
                            Info.LastestIsland = v
                            Info.Number = Do
                        end
                    end
                end
            end
            if not IsRaiding() then
                Info.Ended = true
            end
            return Info
        end
        return Info
    end
    LeaveSpecificPlace = function(name)
        if FirstSea then
            if name == "Skylands" and dist(workspace.Map.SkyArea2.PathwayHouse.Exit.Position) <= 800 then
                
                _C("requestEntrance", workspace.Map.SkyArea1.PathwayTemple.ExitPoint.Position);
            end
            if name == "Underwater City" and dist(workspace.Map.Fishmen.Entrance.Position) <= 5000 then
                
                _C("requestEntrance",Vector3.new(3864.6884765625, 6.736950397491455, -1926.214111328125))
            end
        end
        if SecondSea then
            if name == "Cursed Ship" and dist(workspace.Map.GhostShipInterior["Meshes/BoatFlower_FlowerBoat_05"].Position) <= 5500 then
                
                _C("requestEntrance",Vector3.new(-6508.55810546875, 89.03499603271484, -132.83953857421875))
            end
        end
    end
    tp = function(target,statement,disableIslandSkip,fromRaid,CallingFrom)
        if not Root then return end
        if IsRaiding() and not fromRaid then return end
        if tweenPause then return end
        task.desynchronize()
        local thisId
        local s,e = pcall(function()
            local Dista,distm,middle = dist(target,nil,true),1/0
            if Char and Root and Dista >= 2000 and tick() - recentlySpawn > 5 then
                for i,v in pairs(CanTeleport[SeaIndex]) do
                    local distance = dist(v,target,true)
                    if distance < dist(target,nil,true) and distance < distm then
                        distm,middle = distance,v
                    end
                end
                task.synchronize()
                if middle and InArea(Root.Position) ~= InArea(middle) and not IsRaiding() then
                    -- print(Root.Position,"\n",target.p)
                    -- print(Dista,distm,CurrentArea,InArea(middle))
                    print(CallingFrom)
                    _C("requestEntrance",middle)
                end
                if not disableIslandSkip and Settings.IslandTP then
                    if not IsCombat() and not _hasNonWeapon() and Root and not _hasChip() and (not _hasFruit()) and not IsRaiding() then
                        local Area = InArea(target.p)
                        local MyArea = InArea(Char.HumanoidRootPart.Position)
                        local SpawnPoint = workspace["_WorldOrigin"].PlayerSpawns[Client.Team.Name]:GetChildren()
                        local dista,distm,charDist,nearest = 2000,9000
                        for i,v in pairs(SpawnPoint) do
                            local Position = v:GetPivot().p
                            local distance = dist(target.p,Position,true)
                            if distance <= dista then
                                charDist = dist(Position,nil,true)
                                dista,nearest = distance,v
                            end
                        end
                        if nearest and (charDist <= 8700) then
                            if not Char:FindFirstChild("Humanoid") then return end
                            if not Char:FindFirstChild("HumanoidRootPart") then return end
                            if Char.HumanoidRootPart:FindFirstChild("Died") then
                                Char.HumanoidRootPart.Died:Destroy()
                            end
                            repeat wait()
                                pcall(task.spawn,_C,"SetLastSpawnPoint",nearest.Name)
                            until ClientData.LastSpawnPoint.Value == nearest.Name
                            pcall(function()
                                if Current_Exploit("Fluxus") or Current_Exploit("Comat") or IsValyse then
                                    Char.Humanoid:ChangeState(15)
                                else
                                    Root:Destroy()
                                end
                            end)
                            repeat wait(.1) until Root.Parent
                        end
                    end
                end
                task.desynchronize()
            end
            if tweenActive and lastTweenTarget and (dist(target, lastTweenTarget) < 10 or dist(lastTweenTarget) >= 10) then
                return
            end
            tweenid = (tweenid or 0) + 1 
            lastTweenTarget = target
            statement = statement or function() end
            local Char = Client.Character
            local Root = Char.HumanoidRootPart
            thisId = tweenid
            if Util.FPSTracker.FPS > 60 then
                setfpscap(60)
            end
            task.spawn(pcall,function()
                lastPos = {tick(),target}
                task.synchronize()
                local currentState = statement()
                local currentDistance = dist(Root.Position, target, true)
                local oldDistance = currentDistance
                Char.Humanoid:SetStateEnabled(13,false)
                while Root and currentDistance > 75 and currentState and thisId == tweenid and Char.Humanoid.Health > 0 do
                    local Percent = (58/math.clamp(Util.FPSTracker.FPS,0,60))
                    local Speed = 6*Percent
                    local Current = Root.Position
                    local Dift = Vector3.new(target.X,0,target.Z) - Vector3.new(Current.X,0,Current.Z)
                    local Sx =  (Dift.X < 0 and -1 or 1)*Speed
                    local Sz =  (Dift.Z < 0 and -1 or 1)*Speed
                    local SpeedX = math.abs(Dift.X) < Sx and Dift.X or Sx
                    local SpeedZ = math.abs(Dift.Z) < Sz and Dift.Z or Sz
                    task.spawn(function()
                        currentDistance = dist(Root.Position, target, true)
                        currentState = statement()
                        if currentDistance > oldDistance+10 then
                            tweenid = -1
                            tweenPause = true
                            Root.Anchored = true
                            wait(1)
                            tweenPause = false
                            Root.Anchored = false
                        end
                        oldDistance = currentDistance
                    end)
                    Root.CFrame = Root.CFrame + Vector3.new(math.abs(SpeedZ) < (5*Percent) and SpeedX or SpeedX/1.5, 0, math.abs(SpeedX) < (5*Percent) and SpeedZ or SpeedZ/1.5)
                    Root.CFrame = CFrame.new(Root.CFrame.X,target.Y,Root.CFrame.Z)
                    Char.Humanoid:ChangeState(11)
                    tweenActive = true
                    task.wait(0.001)
                end
                Char.Humanoid:SetStateEnabled(13,true)
                tweenActive = false
                if currentDistance <= 100 and currentState and thisId == tweenid then
                    Root.CFrame = target
                end
            end)
        end)
        if not s then print(e) end
        task.synchronize()
        return thisId
    end
    stoptp = function()
        tweenid = -1
    end
    RaidFruitName = function(txt)
        if txt == "" then
            return "Fruitless"
        end
        local compile = string.match(txt, "%-(.+)")
        if compile then
            return compile
        end
        return txt
    end
    ac_lvl = function()
        task.desynchronize()
        local Level = Client.Data.Level.Value
        local Exp = Client.Data.Exp.Value
        while true do
            local _EXP = Exp_Calc(Level)
            if Exp >= _EXP then
                Level = Level + 1
                Exp = Exp - _EXP
            else
                break
            end
        end
        task.synchronize()
        return math.clamp(Level,0,MAXLEVEL)
    end
    IsRaiding = function()
        return Client.PlayerGui.Main.Timer.Visible or JustStarted
    end
    IsSameName = function(full,sub)
        return full:lower():sub(1,#sub) == sub:lower()
    end
    purchaseCombat = function(name,tf)
        local success,data
        if name == "DragonClaw" then
            success,data = pcall(_C,"BlackbeardReward","DragonClaw","2")
        else
            success,data = pcall(_C,"Buy"..name,tf)
        end
        return success and (data == 1 or data == 2)
    end
