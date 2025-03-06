# UnityGGPO

This is home of UnityGGPO an interop DLL of the ggpo library that works with Unity 3D. This repository is and an amalgam of a few things if you are looking for just the Plugin or the Shared Game Library see Packages section below UPM git url is https://github.com/nykwil/UnityGGPO.git?path=/Unity/Packages/UnityGGPO

## This Fork

This is a fork of the original UnityGGPO repository, with some modifications to enhance the flexibility of GGPO integration.

**Inputs with Custom Sizes**

This fork added several functions to allow inputs of custom size to be passed to `ggpo_add_local_input` and to be retrieved from `ggpo_synchronize_input`. 

In the original UnityGGPO code, sizes of inputs are hardcoded to 64 bits (`uint64_t`), and was designed to be used as a bitfield. This meant that games must have less than or equal to 64 types of unique inputs (which may include variations or states, such as a button being pressed down, held, or released) in order to be able to use UnityGGPO. This fork provided a workaround to this situation by allowing inputs to be passed in as a `byte[]` and to be retrieved as a `byte[][]`, which each `byte[]` representing the input for one player.

Note that the requirements set by GGPO still exists. GGPO requires that the input size to be the same across all calls to `ggpo_add_local_input`, `ggpo_synchronize_input`, and the various begin session functions. It is now up to the developer to ensure that this value is consistent.

This fork also modified the `SynchronizeInput` method (including the newly added method with custom input sizes) to no longer throw an exception on a non-successful status code being returned by GGPO. It now passes that status code out in the `result` output parameter.

Changes made:

- UnityGGPO.h, UnityGGPO.cpp:
    - Added `UggTestStartSessionCustomInputSize(..., const char* game, int num_players, int input_size, int localport))`
    - Added `UggStartSessionCustomInputSize(..., const char* game, int num_players, int input_size, int localport)`
    - Added `UggStartSpectatingCustomInputSize(..., const char* game, int num_players, int input_size, int localport, char* host_ip, int host_port)`
    - Added `UggSynchronizeInputCustomSize(GGPOPtr ggpo, void* inputs, int length, int size, int& disconnect_flags)`
    - Added `UggAddLocalInputCustomSize(GGPOPtr ggpo, int local_player_handle, void* input, int size)`
- GGPO.cs, GGPOSession.cs:
    - Modified `long[] SynchronizeInput` to `long[] SynchronizeInput(IntPtr ggpo, int length, out int result, out int disconnect_flags)`.
    - Added `byte[][] SynchronizeInput(IntPtr ggpo, int length, int size, out int result, out int disconnect_flags)` to retrieve inputs of custom sizes.
    - Added `int AddLocalInput(int local_player_handle, int size, byte[] inputs)` to send inputs of different sizes.
    - Added the relevant start session methods and DLL imports.

**Important** - Note that GGPO still puts a hard limit on the maximum size of inputs in the `GAMEINPUT_MAX_BYTES` macro, which has been modified to 128 (bytes) in this repository. Sending inputs larger than this causes an assertion failure. 

This macro needs to be modified and the UnityGGPO DLL needs to be rebuilt should you wish to expand the input size beyong 128 bytes. Refer to the comment above `game_input.h` and modify the corresponding compiler macros.

## Contents
- A Unity project (/Unity) containing.
  - Package of the plugin wrapper (/Unity/Packages/UnityGGPO)
  - Package for a Shared Game Library (/Unity/Packages/SharedGame)
  - Tests (/Unity/Assets/Tests)
  - VectorWar example with a view like implementation using the Shared Game Library (/Unity/Assets/VectorWar)
  - EcsWar example made with DOTS using the Shared Game Library (/Unity/Assets/EcsWar)
- The CMake project to build the UnityGGPO DLL (/UnityGGPO)
- A submodule of https://github.com/pond3r/ggpo for building the UnityGGPO.dll

## Packages
### Plugin
This is the bare bones wrapper with special Session abstraction layer to make it easier to use then direct DLL calls. Included is only the windows DLLs built from the CMake project found at /UnityGGPO/UnityGGPO

Two ways to use:
- Direct access to the ggpo library using the GGPO class requires unsafe calls and IntPtr function pointers.
- Use the GGPO.Session helper class to have safe access using Unity's native collections library and delegates.

Add to your project in the Package Manager using "Add package from git URL..." with this URL:
https://github.com/nykwil/UnityGGPO.git?path=/Unity/Packages/UnityGGPO

### Share Game Library
This is some boiler plate game layer so that you can create local and multiplayer enabled games with just some simple interfaces implementations. Includes very basic UI and Dialogs for the game. Also has an implementation of the Performance Dialog which you need in some capapcity to run. See the EcsWar of VectorWar examples at https://github.com/nykwil/UnityGGPO. For more information. 

Add to your project in the Package Manager using "Add package from git URL..." with this URL:
https://github.com/nykwil/UnityGGPO.git?path=/Unity/Packages/SharedGame

## Examples and Tests
### VectorWar

VectorWar example using a view like implementation. Found at /Unity/Assets/VectorWar

How to play
- Run the VectorWar.scene.
- Left UI panel is your player index and the connection list.
- Click Start Session

### EcsWar
An example made with DOTS using the Shared Game Library. This is a work in progress. Rollback eventually fails if you are using SharedComponents. Found at /Unity/Assets/EcsWar

## DLL CMake Project

This is the CMake Project found at /UnityGGPO. Windows version only.

Build:
- run build_windows.cmd to build the solution.
- Modify the corresponding compiler macros in `game_input.h` and `bitvector.h` to accomodate for your project's maximum input size.
- build and run the INSTALL project to copy the built DLL into the Unity Plugin folder for the UnityGGPO package.







Feedback always welcome.

TODO
-Shared Game Library needs better documentation.
-Better tests, unit tests, and a way to run sync test.
-ECS rollback needs to be fixed so that it works with SharedComponents
