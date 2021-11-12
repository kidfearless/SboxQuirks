
# [Introduction]

This document is intended to give a quick reference to some of the quirks and features that S&box(sbox from here on) has. Hopefully it will save some headaches in the future when more people start programming more.

Assume most methods are only ran on the server, as that seems to be the default. A notable exception to this is instead the constructor of an Entity. Even if a property is networked and available to the client, the code that references it might not be running on the client.

## [General]
Entities aren't always available on the client, it *might* be due to their transmission state.

There's a `-dedicated` launch option to launch the game as a server, but it doesn't seem to work right now.

A lot of .NET 5 is restricted through a whitelist system. It can be opened up through the whitelist config, but how that will work on the client is yet to be seen.

The `Entity.Scale` property doesn't determine the size of the entity, it more so determines it's mass. see `Entity.PhysicsBody.Mass`. You must change `Entity.LocalScale` in order to change how the client size.

You can access private fields, properties, and methods using reflection, and optimize them by compiling a delegate from `MethodInfo.CreateDelegate()`. Current performance with this is up to 12-24 times slower than accessing a property directely so still avoid it when possible.

You cannot iterate through types in an assembly as the Assembly class is blocked.

You cannot write to files within the addon folder you have to write to the data folder.

## [Networking]

Unmanaged types are the easiest type to network, simply add [Net] to the property and it will be networked. managed types can be networked if they are inside a BaseNetworkable. You should not pass / copy the properties of a BaseNetworkable instance to another. 

## [UI]

There is only one hud entity, it's networked to the client but it isn't per client for all intents and purposes.

Elements aren’t removed on hotload, use my HotloadPanel I made and create your components in the following method `protected  void  InitializeComponents()`

Html templates exist but are fucky, I do not recommend using them.

Images are essentially divs with a background image style applied to it.

A lot of the filters are currently broken.

css calc() is broken.

Radial-gradient isn’t implemented

Linear gradient doesn’t use deg, it uses % and can’t specify stopping points.

  

## [Commands]

Garry in his infinite knowledge has decided to check default keybinds every tick for every player in order to implement commands. Seeing how that’s stupid here is how you implement console input commands. Input commands need to be prefixed with a `+iv_` or `-iv_` in order for the minus command to be sent
  
```c#
// this has to be static, can also pass parameters if they are convertable from a string
[ServerCmd("+iv_dosomething")]  
public static void ServerCommand_Callback()  
{
	// the pawn can sometimes be null here just like how the client can be 0 in SM
	if(ConsoleSystem.Caller.Pawn is not KFPlayer player)
	{
		return;
	}  
	
	// this will call it on both the server and the client instantly if it's rpc'd
	player.DoSomething();
}

[ServerCmd("-iv_dosomething")]  
public static void ServerCommand_Callback()  
{
	if(ConsoleSystem.Caller.Pawn is not KFPlayer player)
	{
		return;
	}  
	player.DoSomethingMinus();
}
//... inside the player class

// this isn't required to be static
[ClientRPC]
public void DoSomething(){}
[ClientRPC]
public void DoSomethingMinus(){}
```

Only clients can process `Input.GetKeyWithBinding`, specifically client commands and client menus

## [Movement]
Movement vectors are no longer on a scale of -450 to 450. Instead are in a scale of -1 to 1.

Angles are no longer stored as a QAngle/Vector3 but instead are now a Quaternion/Rotation object. You can use `EyeRot.Angles();` to access them as an angle and the `Rotation.From( myAngle );` to assign the rotation back.
