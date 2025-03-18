
Alchemy is a potential replacement for Lua, C#, or python in your next game project. 

Inspired by the popularity of ECS designs, the simplicity of Lua, and the concurrency of Skookum Script, Alchemy is a toolbox for game makers. 

Alchemy Features: 

First Class ECS Support: 
 All functions operate as queries when applied to entity collections: 
 
 '''
 Mod MyGame
 Pos := Vec2
 Health := Dec
 Name := Str
 Player := Ent(Pos,Health,Name)

 MoveTo := Fn(pos:Pos,health:Health,name:Name,newPos:Pos > Pos,Health,Name) 
             {
                newPos,health,name
             }
 Main()
 {
    World :[]Player = [{.Pos = {}, .Health = 100, .Name = "Will"}, {.Pos = {0,-100},.Name = "UI"}]

    World.= World.map(MoveTo(.newPos = {10,10})) -- moves "Will" to 10,10, leaves "UI" unchanged

 }
 '''

 Frictionless C-Interop
 '''
 CMod cRenderer
 Box := Struct{x:Dec,y:Dec,w:Dec,h:Dec}
 Color := Vec3
 opaquePointer := CType(8) -- declare C types which may not be operated on in Alchemy, but can be passed around
 render := Fn(rect:Box,color:Color > opaquePointer); -- this will create a function in <ProjectName>/C/cRenderer.h called "cRenderer_render(const cRenderer_Box* box, const cRenderer_Color* color)"
 

 Mod MyGame

 Main()
 {
    while(true)
    {
        render({0,0,100,100},{255,255,255}) -- draws a white box in the top left corner using any existing c renderer
    }

 }

 '''

 Intuitive and foot-gun free concurrency:
'''
 Main()
 {
    start := Vec2{0,0}
    end := Vec2{0,10}
    leyline -- declares a section in which all assignments and argument passes form a dependency graph which is respected by the runtime's threadpool
    {
        white := Box{.w = 100, .h = 100}
        overTime(3.0) -- defines implicit local variables dt, and t and tEnd, fracDone
        {
            white[x,y] = Lerp(start,end,fracDone)  
        }  
        green := Box{.w = 100, .h = 100}
        overTime(10.0)
        {
            green[x,y] = Lerp(end,start,fracDone)  
        }
        -- over three seconds a white box will move from 0,0 to 0,10
        -- over ten seconds a green box will move from 0,0 to 0,10
        -- these operations begin at the same time
        while(live)
        {
            render(white,Color{255,255,255}) 
            render(green,Color{0,255,0})
        }
    }
    -- outside of a leyline block each coroutine waits to return before moving on to the next 
    white := Box{.w = 100, .h = 100}
    overTime(3.0) -- defines implicit local variables dt, and t and tEnd, fracDone
    {
        white[x,y] = Lerp(start,end,fracDone)  
        render(white,Color{255,255,255}) -- draws a white square which moves from 0,0 to 0,10 over three seconds
    }  
    green := Box{.w = 100, .h = 100}
    overTime(10.0)
    {
        green[x,y] = Lerp(end,start,fracDone)  
        render(green,Color{0,255,0}) -- draws a green square which moves from 0,10 to 0,0 over ten seconds
    }

    -- will wait here until everything finishes 
    
}
 '''


Slot into any project as "game-play shaders" 
'''
Mod CallMe
double := CType(8)
GameData := Struct
{
    player0 : Player;
    player1 : Player;
    heightMap := [2,?]double; -- a 2-D dynamic array of doubles
}

Main(data:GameData > data ) -- adding arguments and return values to main enables the calling of an alchemy program as a c-function
{
    data.player0.=add(4) -- add four to the player's positions
    data.heightMap.resize(2,2000)
}

 // my_existing_project.c
#include"alch.h"
void main()
{
    AlchInstance instance;
    AlchInstance_Init(&instance);
    AlchProgram* CallMe := AlchInstance_LoadModule(&instance,"CallMe")
    CallMe_GameData data;
    while(true)
    {
        ALCH_CALL_MAIN(CallMe,&data,CallMe_GameData); -- performs a checked call on the main function in the CallMe module
    }
}
'''
