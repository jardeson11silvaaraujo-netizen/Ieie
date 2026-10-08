-- EGG KAITUN V115 - V114 BASE + ROLLBACK SHIELD 60FPS
-- Pickup forensic preservado.
-- One death = STOP.
-- Flight: collision-aware route + passive rollback shield, 60Hz Heartbeat telemetry.
-- IMPORTANTE: o shield detecta/cancela rollback; nao tenta brigar com a autoridade fisica.

local Players=game:GetService("Players")
local RS=game:GetService("ReplicatedStorage")
local WS=game:GetService("Workspace")
local RunService=game:GetService("RunService")
local TweenService=game:GetService("TweenService")
local PathfindingService=game:GetService("PathfindingService")
local HttpService=game:GetService("HttpService")
local LP=Players.LocalPlayer

local CFG={
 Chicken=Vector3.new(545,71,-365),
 ForestTrigger=Vector3.new(592.974,70.683,-332.401),
 ChickenSpeed=120, FlightSpeed=1000,
 ChickenAfterWait=.06, ForestAfterTPWait=.12, AskPostWait=.70,
 ForestPromptOffsets={0,.067143,.114738},
 BeginTimeout=6, EggTPHeight=3,
 PromptLeadBeforeEnd=.159, EscapeAfterLastPrompt=.180,
 TargetPromptOffsets={0,.055519,.208221,.292291,.358595,.414449,.508160,.610433,.671248,.778416, .988710,1.058775,1.157651,1.310894},
 TargetPromptSearchRadius=18, PickupToFlightDelay=.12, BaseOffsetY=2.5, BaseAirHeight=28.3,
 BaseHorizontalArrival=32, BaseFallTimeout=3.2, BaseAirArrival=20,
 RollbackJumpDistance=220, RollbackDistanceIncrease=170,
 FlightWaypointSpacing=120, FlightPathAgentRadius=2.5, FlightPathAgentHeight=6, FlightPathWaypointGap=4,
 RagdollDurationFallback=2.5,
 AutoRepeat=true,CycleDelay=.30,ReportMovementInterval=1/60,MaxMovementSamples=500
}

local S={Running=true,Dead=false,Character=nil,Humanoid=nil,Root=nil,CurrentTween=nil,
 BasePlot=nil,BaseReturn=nil,Mode="INIT",Target=nil,PendingAuth=false,AuthDone=false,
 AuthTargetPosition=nil,AuthBeginAt=0,AuthDuration=2.5,Ragdoll=false,RagdollCount=0,
 FirstBeginAt=0,FirstEndAt=0,CachedTargetPrompt=nil,PromptFireCount=0,
 FlightActive=false,FlightInterrupted=false,FlightRollback=false,FlightSuccess=false,
 FlightFailure=nil,FlightGeneration=0,AskDuration=0,AskResult=nil,Relocates=0,
 Events={},Movement={},StartClock=os.clock(),LastArea=nil}

local function now() return os.clock()-S.StartClock end
local function vec(v) if typeof(v)~="Vector3" then return "nil" end return string.format("(%.3f, %.3f, %.3f)",v.X,v.Y,v.Z) end
local function alive() return not S.Dead and S.Character and S.Character.Parent and S.Humanoid and S.Humanoid.Parent and S.Humanoid.Health>0 and S.Root and S.Root.Parent end
local function state() if not S.Humanoid then return "nil" end local ok,v=pcall(function() return S.Humanoid:GetState() end) return ok and tostring(v) or "nil" end
local function area()
 for _,h in ipairs({LP,S.Character}) do
  if h then for _,k in ipairs({"AreaId","Area","CurrentArea","Zone","ZoneId"}) do
   local ok,v=pcall(function() return h:GetAttribute(k) end)
   if ok and v~=nil then return tostring(v) end
  end end
 end
 return "nil"
end
local function context()
 if not alive() then return "dead/root=nil" end
 return string.format("pos=%s | area=%s | state=%s | speed=%.1f | hp=%.1f | rag=%s",vec(S.Root.Position),area(),state(),S.Root.AssemblyLinearVelocity.Magnitude,S.Humanoid.Health,tostring(S.Ragdoll))
end
local function log(n,d)
 d=tostring(d or "")
 S.Events[#S.Events+1]={t=now(),name=n,detail=d}
 print(string.format("[V115 %.6f] %-28s | %s",now(),n,d))
end
local function report()
 local l={"==========================================================================" ,"V115 - V114 BASE + ROLLBACK SHIELD 60FPS","==========================================================================" ,
 string.format("Duration: %.6f",now()),"Final: "..context(),
 string.format("Mode=%s | Dead=%s | Relocates=%d | Ragdolls=%d | AskDuration=%.6f | AskResult=%s | PromptFires=%d",
 S.Mode,tostring(S.Dead),S.Relocates,S.RagdollCount,S.AskDuration,tostring(S.AskResult),S.PromptFireCount)}
 if S.Target then l[#l+1]=string.format("TARGET uid=%s | asset=%s | rate=%s | area=%s | pos=%s",S.Target.Key,S.Target.Asset,S.Target.Rate,S.Target.Area,vec(S.Target.Position)) end
 l[#l+1]="";l[#l+1]="TIMELINE"
 for _,e in ipairs(S.Events) do l[#l+1]=string.format("[%.6f] %-28s | %s",e.t,e.name,e.detail) end
 l[#l+1]="";l[#l+1]="MOVEMENT"
 for i,m in ipairs(S.Movement) do l[#l+1]=string.format("#%04d [%.6f] pos=%s | area=%s | state=%s | speed=%.1f | mode=%s",i,m.t,vec(m.pos),m.area,m.state,m.speed,m.mode) end
 l[#l+1]="==========================================================================" ;l[#l+1]="END V115"
 return table.concat(l,"\n")
end
local function copyReport()
 local r=report();print(r)
 if type(setclipboard)=="function" then pcall(setclipboard,r) elseif type(toclipboard)=="function" then pcall(toclipboard,r) end
end

pcall(function()
 local pg=LP:FindFirstChild("PlayerGui")
 if pg then local x=pg:FindFirstChild("EggKaitunV110");if x then x:Destroy()end end
end)

local Gui=Instance.new("ScreenGui");Gui.Name="EggKaitunV115";Gui.ResetOnSpawn=false;Gui.IgnoreGuiInset=true;Gui.Parent=LP:WaitForChild("PlayerGui"
