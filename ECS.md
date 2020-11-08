I'm going to show you how to write a game from dead scratch with as little help
as possible. I belive the best way to learn anything like this is to just do it
yourself with as little help as possible, and fall into every single pitfall and
trap. You'll learn things the hard way, but you'll remember and understand them
much better than if you copy and paste thousands of lines of code from a book or
website somewhere.

I'm writing this because when I first tried to write a game from scratch, I read
a bunch of tutorials about writing engines and they provided way too much
detail. They used all sorts of STL containers in all sorts of fancy ways, and
were pretty confusing. The issue with reading someone's verbose game engine
tutorial is that they have all sorts of advanced edge cases and performance
optimizations in mind. That's great, but it obfuscates the core thing we're
trying to learn: how the hell a game works inside.

The goal of this article is to provide you with the smallest possible amount of
information you need to go play around on your own and figure out on your own
how to *really* write a game engine. I'm going to describe the simplest possible
**ECS (Entity-Component-System)** game engine. I'll explain what those terms
mean as we go.

Before I begin, I will present the whole thing up front so you can see what
we're diving into:

```cpp
// ### Define Entity type ###
using Entity = uint32_t;


// ### Define Component types ###

using Position = glm::vec3;

using Velocity = glm::vec3;

struct Drawable
{
	Mesh* m_mesh;
	std::vector<Texture*> m_textures;
	Shader* m_shader;
};

// ...

// ### Define component tags ###
using Tag = uint32_t;
namespace CompTags
{
	constexpr Tag NullTag = 0x0;
    constexpr Tag Position = 0x1;
    constexpr Tag Velocity = 0x1 << 1;
    constexpr Tag Drawable = 0x1 << 2;
    // ...
}

// ### Define game data arrays ###

constexpr int N_ENTS = 100;

namespace GameData
{
    std::array<Tag, N_ENTS> tags;

    std::array<Position, N_ENTS> positions;
    std::array<Velocity, N_ENTS> velocities;
    std::array<Drawable, N_ENTS> drawables;
    // ...
}

// ### Define game systems ###

// Update positions from velocities
void process_kinematics(float dt)
{
    constexpr Tag rqdTags = CompTags::Position | CompTags::Velocity;
    for (Entity e = 0; e < N_ENTS; e++)
    {
        if ((GameData::tags[e] & rqdTags) != rqdTags) { continue; }

        Velocity& v = GameData::velocities[e];
        Position& p = GameData::positions[e];

        p += v*dt;
    }
}

// Draw drawable entities
void draw()
{
	for (Entity e = 0; e < N_ENTS; e++)
	{
		if (!(GameData::tags[e] & CompTags::Drawable)) { continue; }

		// Draw the entity
	}
}


// ### Main game loop ###

int main()
{
	constexpr float dt = 0.01f;
	bool runGame = true;

	while (runGame)
	{
		process_kinematics(dt);
		draw();
	}

	return 0;
}
```

Let's just dive straight in:

```cpp
// ### Define Entity type ###
using Entity = uint32_t;
```
Everything in your game will be an entity. The player will be an entity, their
vehicle will be, the bullets they shoot will be, etc. The `Entity` type itself
is nothing but an integer ID. The actual behaviors and properties of each entity
are encoded in the form of **components**.

```cpp
// ### Define Component types ###

using Position = glm::vec3;

using Velocity = glm::vec3;

struct Drawable
{
    Mesh* m_mesh;
    std::vector<Texture*> m_textures;
    Shader* m_shader;
};

// ...
```

Components represent behaviors that entities may express. Any entity that
exists at some position in the game world (as opposed to being some sort of
abstract object) would have a `Position` component (in this case represented by
a GLM 3-vector, but you can use whatever you like). Any object that moves would
have a `Velocity` component. Any object that has a graphical appearance would
have a `Drawable` component. I'll explain how components are associated with
entities in a bit.

This process of describing objects by associating them with various properties
is called "composition" and is very handy. Here are some examples of how you
might start building various game objects via composition:

 * An on-screen waypoint marker, for instance, could be made from an entity with
 a `Position` and a `Drawable`, and placed at the waypoint location

 * A trigger (that detects if a player enters an area) would have a `Position`
 but no `Drawable` (since it has no visible appearance). Instead, it would
 probably have some sort of `Trigger` component that handles the callback that
 is executed when the trigger is tripped

 * Say you have a game with boats and cars. If you wanted to make an amphibious
 vehicle, rather than having to code some custom logic for it, you just give it
 both a `Floats` and `Drives` component, and it will automatically be handled
 by the game logic that is responsible for those behaviors

 * A bullet fired by the player would certainly have a `Position` and
 `Velocity`, but probably wouldn't have a `Drawable` unless it's some large
 projectile that one might expect to be visible to the naked eye in-flight. It
 would probably also have a `Collidable` or `Projectile` component or some other
 way to remind us to check nearby objects to see if it's hit anything. You'd
 spawn these entities at the muzzle of a gun with the appropriate velocity
 vector and let the physics take care of the rest

At the end, I'll give a list of suggested components that might be useful.

Components should be POD structs only, if possible. The behavior itself will be
defined in **systems** (which I'll describe shortly), so the components only
need to store the data relevant to their behavior, not account for any logic.
You'll see.

Now we need a way to associate entities with their component data:

```cpp
using Tag = uint32_t;
namespace CompTags
{
	constexpr Tag NullTag = 0x0;
    constexpr Tag Position = 0x1;
    constexpr Tag Velocity = 0x1 << 1;
    constexpr Tag Drawable = 0x1 << 2;
    // ... etc
}
```

An easy way to attach components to entities is using bitflags. Each entity will
have a `Tag` object that we do bitwise ops on in order to add, remove, or test
for the presence of the components. Finally, our actual game object data will be
stored in plain old arrays:

```cpp
constexpr int N_ENTS = 100;

namespace GameData
{
    std::array<Tag, N_ENTS> tags;

    std::array<Position, N_ENTS> positions;
    std::array<Velocity, N_ENTS> velocities;
    std::array<Drawable, N_ENTS> drawables;
    // ...
}
```

We store an array `GameData::tags` which stores the `Tag` for each entity.
Note that since `Entity` is just an int ID, we don't even need to store them
anywhere. The `Entity` ID is implied by the array index of the tag or component.

The rest of the arrays store our actual component data. Let's say that we want
to get the position of entity #5. We can do:

```cpp
Entity e = 5;
if (GameData::tags[e] & CompTags::Position)
{
    Position& p = GameData::positions[e];
}
```
Pretty straightforward, right? The game object, whatever it is, is specified by
a unique ID `5`, which is also used to index into the component and tag arrays
to find the data that belongs to it. It's not a very efficient configuration:
say you have some very specialized component, such as a "`MissileGuidance`"
comp. If this component is only given to guided missiles that are actively
in-flight, there will probably only ever be a few slots of the array actually
being used at any given time, but that's the price we pay for our exceedingly
simple arrangement. If that displeases you, go improve it yourself :P

The actual game logic and component behaviors are implemented as **systems**,
which all follow a similar pattern:

```cpp
// Draw drawable entities
void draw()
{
	for (Entity e = 0; e < N_ENTS; e++)
	{
		if (!(GameData::tags[e] & CompTags::Drawable)) { continue; }

		// Draw the entity
	}
}
```
A system loops over all entities, and checks the `tags` array to see if the
current entity is possesses the components it operates on. If the entity has
the right component (in this case `Drawable`), the system performs its logic
on the entity/component; if the component is missing, the entity is skipped.

Here's another system, fully implemented:

```cpp
// Update positions from velocities
void process_kinematics(float dt)
{
    constexpr Tag rqdTags = CompTags::Position | CompTags::Velocity;
    for (Entity e = 0; e < N_ENTS; e++)
    {
        if ((GameData::tags[e] & rqdTags) != rqdTags) { continue; }

        Velocity& v = GameData::velocities[e];
        Position& p = GameData::positions[e];

        p += v*dt;
    }
}
```

This one updates object positions. It operates on `Position` and `Velocity`, so
it pre-defines a `rqdTags` variable storing the bitwise OR of the corresponding
tags. As it loops, it checks to see if `(tag & rqdTags) == rqdTags` (if not, the
entity is missing one or more of the required components and is skipped). If the
entity meets the requirements, the system grabs a reference to the entity's
position and velocity components, and updates the position via simple Euler
integration.

The modularity of ECS is great because it makes it easy to keep logic decoupled.
You can be adding some weird new object or vehicle to your game, and even if it
needs some sort of new special logic to determine how it accelerates, most of its
behaviors will probably be covered by the basic systems you added previously,
such as this kinematics one.

```cpp
int main()
{
	constexpr float dt = 0.01f;
	bool runGame = true;

	while (runGame)
	{
		process_kinematics(dt);
		draw();
	}

	return 0;
}
```

Finally, we run our game loop in `main()`. The real thing will be much more
complicated, as you'll have many systems to update, GLFW/SDL events to process,
and more, but you get the idea. Here we use a fixed timestep of 0.01 seconds
(equivalent to 100FPS), but you'll probably want to record wall clock time and
pass in the true delta-time so that the game runs at a fixed speed rather than
being dependent on framerate.

There you go, an engine skeleton in less than 100 lines of code. Obviously you
need to actually implement and add tons more components and systems. This
includes the actual `draw()` system that performs the rendering logic. OpenGL
is a whole can of worms by itself, but as you go through
[learnopengl.com](https://learnopengl.com/), keep this game engine structure in
mind; although some of the setup/loading stuff will happen outside of the ECS
structure, the actual object-specific graphics data will need to be integrated
into whatever arrangement you devise to handle renderable objects.

That's about it. Knock yourself out.

Here's a list of basic components to get you thinking:

| Components           | Function                                                   |
|----------------------|------------------------------------------------------------|
| Position             | self explanatory                                           |
| Velocity             |                                                            |
| Acceleration         |                                                            |
| Orientation          |                                                            |
| Angular Velocity     |                                                            |
| Angular Acceleration |                                                            |
| Mass                 |                                                            |
| Timer                | Triggers a callback when its internal countdown completes  |
| Drawable             | Stores mesh, textures, shaders, etc used to draw an object |
| Camera               | Represents a 3D viewport camera, used by rendering system  |
| AttachedTo           | Physically attaches one entity to another                  |
| Health               | Keeps track of an entity's health/hitpoints                |
| Armor                | Keeps track of an entity's armor status/points             |