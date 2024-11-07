//CS:S Goremod v5.3.2 by Joe 'DiscoBBQ' Maley:

//Terminate:
#pragma semicolon 1

//Includes:
#include <sourcemod>
#include <sdktools>

//Overflow:
static Float:PrethinkBuffer;

//Weapons:
static String:CSWeapon[7][32] = {"m3",
						"xm1014",
						"hegrenade",
						"g3sg1",
						"sg550",
						"awp",
						"scout"
};

//Convars:
new Handle:hBleedingFreq;
new Handle:hImpactBlood;

//Misc:
new BloodClient[2000];

//Information:
public Plugin:myinfo =
{

	//Initialize:
	name = "Realistic Blood",
	author = "+SyntX (Basec on CSSGore)",
	description = "Changed the color of blood to green from red.",
	version = "5.3.2",
	url = "https://steamcommunity.com/id/syntx34"
}

//Parent to Dead Body:
ParentToBody(Client, Particle, bool:Headshot = true)
{

	//Client:
	if(IsClientConnected(Client))
	{

		//Declare:
		decl Body;
		decl String:tName[64], String:Classname[64], String:ModelPath[64];

		//Initialize:
		Body = GetEntPropEnt(Client, Prop_Send, "m_hRagdoll");

		//Edict:
		if(IsValidEdict(Body))
		{

			//Target Name:
			Format(tName, sizeof(tName), "Body%d", Body);

			//Find:
			GetEdictClassname(Body, Classname, sizeof(Classname));

			//Model:
			GetClientModel(Client, ModelPath, 64);

			//Body Exists:
			if(IsValidEntity(Body) && StrEqual(Classname, "cs_ragdoll", false))
			{

				//Properties:
				DispatchKeyValue(Body, "targetname", tName);
				GetEntPropString(Body, Prop_Data, "m_iName", tName, sizeof(tName));

				//Parent:
				SetVariantString(tName);
				AcceptEntityInput(Particle, "SetParent", Particle, Particle, 0);
				if(Headshot)
				{
			
					//Work-Around:
					if(StrContains(ModelPath, "sas", false) != -1 || StrContains(ModelPath, "gsg", false) != -1)
					{

						//Back:
						SetVariantString("primary");
					}
					else
					{

						//Head:
						SetVariantString("forward");
					}

				}
				else
				{
			
					//Back:
					SetVariantString("primary");
				}

				//Parent:
				AcceptEntityInput(Particle, "SetParentAttachment", Particle, Particle, 0);
			}
		}
	}
}

//Write:
//Write:
WriteParticle(Ent, String:ParticleName[], bool:Death = false, bool:Headshot = false)
{
    // Check if entity is a Terrorist (Team 2)
    if (GetEntProp(Ent, Prop_Send, "m_iTeamNum") != 2) {
        return; // Exit function if not a Terrorist
    }

    //Declare:
    decl Particle;
    decl String:tName[64];

    //Initialize:
    Particle = CreateEntityByName("info_particle_system");

    //Validate:
    if (IsValidEdict(Particle))
    {
        //Declare:
        decl Float:Position[3], Float:Angles[3];

        //Initialize Angles for particle orientation:
        Angles[0] = GetRandomFloat(0.0, 360.0);
        Angles[1] = GetRandomFloat(-15.0, 15.0);
        Angles[2] = GetRandomFloat(-15.0, 15.0);

        //Origin:
        GetEntPropVector(Ent, Prop_Send, "m_vecOrigin", Position);

        // Adjust Z-axis position based on Death flag
        if (Death) {
            Position[2] += GetRandomFloat(0.0, 5.0);
        } else {
            Position[2] += GetRandomFloat(15.0, 35.0);
        }

        // Set particle position and orientation
        TeleportEntity(Particle, Position, Angles, NULL_VECTOR);

        // Target Name:
        Format(tName, sizeof(tName), "Entity%d", Ent);
        DispatchKeyValue(Ent, "targetname", tName);
        GetEntPropString(Ent, Prop_Data, "m_iName", tName, sizeof(tName));

        // Particle properties
        DispatchKeyValue(Particle, "targetname", "CSSParticle");
        DispatchKeyValue(Particle, "parentname", tName);
        DispatchKeyValue(Particle, "effect_name", ParticleName);

        // Spawn the particle system
        DispatchSpawn(Particle);

        // Parent the particle to specific attachment for entire body effect
        if (Death) {
            if (Headshot) {
                ParentToBody(Ent, Particle, true); // Attach to head for headshot
            } else {
                ParentToBody(Ent, Particle, false); // Attach centrally for body shots
            }
        } else {
            // For non-death particles, attach particle to a central point on the entity
            SetVariantString("forward");  // Attaching at a forward position on the entity
            AcceptEntityInput(Particle, "SetParentAttachment", Particle, Particle, 0);
        }

        // Activate particle system and start particle effect
        ActivateEntity(Particle);
        AcceptEntityInput(Particle, "start");

        // Timer to delete the particle entity after it finishes
        CreateTimer(3.0, DeleteParticle, Particle);
    }
}



//Delete:
public Action:DeleteParticle(Handle:Timer, any:Particle)
{

	//Validate:
	if(IsValidEntity(Particle))
	{

		//Declare:
		decl String:Classname[64];

		//Initialize:
		GetEdictClassname(Particle, Classname, sizeof(Classname));

		//Is a Particle:
		if(StrEqual(Classname, "info_particle_system", false))
		{

			//Delete:
			RemoveEdict(Particle);
		}
	}
}

//Decal:
stock Decal(Client, Float:Direction[3], bool:Bleeding = false)
{

	//Declare:
	decl Blood;
	decl String:Angles[128];

	//Format:
	Format(Angles, 128, "%f %f %f", Direction[0], Direction[1], Direction[2]);

	//Blood:
	Blood = CreateEntityByName("env_blood");

	//Create:
	if(IsValidEdict(Blood))
	{

		//Spawn:
		DispatchSpawn(Blood);

		//Properties:
		DispatchKeyValue(Blood, "color", "0");
		DispatchKeyValue(Blood, "amount", "1000");
		DispatchKeyValue(Blood, "spraydir", Angles);
		DispatchKeyValue(Blood, "spawnflags", "12");

		//Timer:
		if(!Bleeding)
		{

			//Save & Send:
			BloodClient[Blood] = Client;
			CreateTimer(GetRandomFloat(0.1, 1.0), EmitBlood, Blood);
		} 
		else AcceptEntityInput(Blood, "EmitBlood", Client);
	}

	//Detatch:
	if(Bleeding && IsValidEdict(Blood)) RemoveEdict(Blood);
}

//Emit Blood:
public Action:EmitBlood(Handle:Timer, any:Blood)
{

	//Emit:
	if(IsValidEdict(Blood) && IsClientConnected(BloodClient[Blood]))
		AcceptEntityInput(Blood, "EmitBlood", BloodClient[Blood]);

	//Detatch:
	if(IsValidEdict(Blood)) 
		RemoveEdict(Blood);
}

//Direction:
stock CalculateDirection(Float:ClientOrigin[3], Float:AttackerOrigin[3], Float:Direction[3])
{

	//Declare:
	decl Float:RatioDiviser, Float:Diviser, Float:MaxCoord;

	//X, Y, Z:
	Direction[0] = (ClientOrigin[0] - AttackerOrigin[0]) + GetRandomFloat(-25.0, 25.0);
	Direction[1] = (ClientOrigin[1] - AttackerOrigin[1]) + GetRandomFloat(-25.0, 25.0);
	Direction[2] = (GetRandomFloat(-125.0, -75.0));

	//Greatest Coordinate:
	if(FloatAbs(Direction[0]) >= FloatAbs(Direction[1])) MaxCoord = FloatAbs(Direction[0]);
	else MaxCoord = FloatAbs(Direction[1]);

	//Calculate:
	RatioDiviser = GetRandomFloat(100.0, 250.0);
	Diviser = MaxCoord / RatioDiviser;
	Direction[0] /= Diviser;
	Direction[1] /= Diviser;

	//Close:
	return;
}

//Gib:
stock Gib(Float:Origin[3], Float:Direction[3], String:Model[])
{

	//Declare:
	decl Ent, Roll;
	decl Float:MaxEnts;
	decl Float:Velocity[3];

	//Initialize:
	Ent = CreateEntityByName("prop_physics");
	MaxEnts = (0.9 * GetMaxEntities());
	Velocity[0] = Direction[0] * 400.0;
	Velocity[1] = Direction[0] * 400.0;
	Velocity[2] = Direction[0] * 400.0;
		
	//Anti-Crash:
	if(Ent < MaxEnts)
	{

		//Properties:
		DispatchKeyValue(Ent, "model", Model);
		SetEntProp(Ent, Prop_Send, "m_CollisionGroup", 1); 

		//Spawn:
		DispatchSpawn(Ent);
		
		//Send:
		TeleportEntity(Ent, Origin, Direction, Velocity);

		//Blood:
		Roll = GetRandomInt(1, 5);

		//blood_advisor_pierce_spray:
		if(Roll == 1) WriteParticle(Ent, "blood_advisor_pierce_spray");

		//blood_advisor_pierce_spray_b:
		if(Roll == 2) WriteParticle(Ent, "blood_advisor_pierce_spray_b");

		//blood_advisor_pierce_spray_c:
		if(Roll == 3) WriteParticle(Ent, "blood_advisor_pierce_spray_c");

		//blood_zombie_split_spray:
		if(Roll == 4) WriteParticle(Ent, "blood_zombie_split_spray");

		//blood_advisor_pierce_spray:
		if(Roll == 5) WriteParticle(Ent, "blood_advisor_pierce_spray");

		//Delete:
		CreateTimer(GetRandomFloat(15.0, 30.0), RemoveGib, Ent);
	}
}


//Random Gib:
stock RandomGib(Float:Origin[3], String:Model[])
{

	//Declare:
	decl Float:Direction[3];

	//Origin:
	Origin[0] += GetRandomFloat(-10.0, 10.0);
	Origin[1] += GetRandomFloat(-10.0, 10.0);
	Origin[2] += GetRandomFloat(-20.0, 20.0);

	//Direction:
	Direction[0] = GetRandomFloat(-1.0, 1.0);
	Direction[1] = GetRandomFloat(-1.0, 1.0);
	Direction[2] = GetRandomFloat(150.0, 200.0);

	//Gib:
	Gib(Origin, Direction, Model);
}

//Remove Gib:
public Action:RemoveGib(Handle:Timer, any:Ent)
{

	//Declare:
	decl String:Classname[64];

	//Initialize:
	if(IsValidEdict(Ent))
	{

		//Find:
		GetEdictClassname(Ent, Classname, sizeof(Classname)); 

		//Kill:
		if(StrEqual(Classname, "prop_physics", false)) RemoveEdict(Ent);
	}
}

//Body:
stock RemoveBody(Client)
{

	//Declare:
	decl BodyRagdoll;
	decl String:Classname[64];

	//Initialize:
	BodyRagdoll = GetEntPropEnt(Client, Prop_Send, "m_hRagdoll");
	if(IsValidEdict(BodyRagdoll))
	{
	
		//Find:
		GetEdictClassname(BodyRagdoll, Classname, sizeof(Classname)); 

		//Remove:
		if(StrEqual(Classname, "cs_ragdoll", false)) RemoveEdict(BodyRagdoll);
	}
}

// Bleed:
public Action:Bleed(Handle:Timer, any:Client)
{
    // Check if the client is connected, in game, and on the Terrorist team (team 2)
    if (IsClientInGame(Client) && GetEntProp(Client, Prop_Send, "m_iTeamNum") == 2)
    {
        // Check if client is hurt and still alive
        if (GetClientHealth(Client) < 100 && IsPlayerAlive(Client))
        {
            // Declare:
            decl Roll;
            decl Float:Origin[3], Float:Direction[3];

            // Initialize:
            Roll = GetRandomInt(1, 2);
            GetClientAbsOrigin(Client, Origin);
            Origin[2] += 10.0;

            // Green Blood Splatter:
            if (Roll == 1) WriteParticle(Client, "blood_impact_green_01");
            if (Roll == 2) WriteParticle(Client, "blood_impact_green_01_droplets");

            // Direction for Decal:
            Direction[2] = -1.0;
            Decal(Client, Direction, true);
        }
    }
}



//Damage:
public EventDamage(Handle event, const String:Name[], bool:Broadcast)
{
    // Declare:
    decl Client, Roll;
    decl Float:Origin[3];

    // Initialize:
    Client = GetClientOfUserId(GetEventInt(event, "userid"));

    // Check if the client is on the Terrorist team (team 2)
    if (GetEntProp(Client, Prop_Send, "m_iTeamNum") == 2)
    {
        GetClientAbsOrigin(Client, Origin);
        Origin[2] += 35.0;

        // Multiplier:
        for (new X = 0; X < GetConVarInt(hImpactBlood); X++)
        {
            // Only Green Blood Effects:
            
            // blood_impact_green_01:
            Roll = GetRandomInt(1, 2);
            if (Roll == 1) WriteParticle(Client, "blood_impact_green_01");

            // blood_impact_green_01_droplets:
            Roll = GetRandomInt(1, 2);
            if (Roll == 2) WriteParticle(Client, "blood_impact_green_01_droplets");
        }
    }
}



//Pre-Think:
public OnGameFrame()
{

	//Think Overflow:
	if(PrethinkBuffer <= (GetGameTime() - GetConVarInt(hBleedingFreq)))
	{

		//Refresh:
		PrethinkBuffer = GetGameTime();

		//Declare:
		decl MaxPlayers;

		//Initialize:
		MaxPlayers = GetMaxClients();
	
		//Loop:
		for(new Client = 1; Client <= MaxPlayers; Client++)
		{

			//Connected:
			if(IsClientInGame(Client))
			{

				//Alive:
				if(IsPlayerAlive(Client))
				{
					CreateTimer(1.0, Bleed, Client);
				}
			}
		}
	}
}

//Precache:
ForcePrecache(String:ParticleName[])
{

	//Declare:
	decl Particle;
	
	//Initialize:
	Particle = CreateEntityByName("info_particle_system");
	
	//Validate:
	if(IsValidEdict(Particle))
	{

		//Properties:
		DispatchKeyValue(Particle, "effect_name", ParticleName);
		
		//Spawn:
		DispatchSpawn(Particle);
		ActivateEntity(Particle);
		AcceptEntityInput(Particle, "start");
		
		//Delete:
		CreateTimer(0.3, DeleteParticle, Particle, TIMER_FLAG_NO_MAPCHANGE);
	}
}

//Map Start:
public OnMapStart()
{
	ForcePrecache("blood_impact_green_01");
    ForcePrecache("blood_impact_green_01_droplets");
}

//Initation:
public OnPluginStart()
{

	//Events:
	HookEvent("player_hurt", EventDamage);

	//Server Variable:
	CreateConVar("sm_gblood_version", "1.1", "Gore Blood Version", FCVAR_SPONLY|FCVAR_UNLOGGED|FCVAR_DONTRECORD|FCVAR_REPLICATED|FCVAR_NOTIFY);

	//Convars:
	hBleedingFreq = CreateConVar("sm_gblood_bleeding_frequency", "3", "Time between bleeding.");
	hImpactBlood = CreateConVar("sm_gblood_impact_blood_multiplier", "10", "Multiplier for amount of extra blood on normal impact damage.");
}
