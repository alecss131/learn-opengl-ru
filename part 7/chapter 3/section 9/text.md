# Audio

The game's making great progress, but it still feels a bit empty as there's no audio whatsoever. In this chapter we're going to fix that.

OpenGL doesn't offer us any support for audio capabilities \(like many other aspects of game development\). We have to manually load audio files into a collection of bytes, process and convert them to an audio stream, and manage multiple audio streams appropriately for use in our game. This can get complicated pretty quick and requires some low-level knowledge of audio engineering.

If it is your cup of tea then feel free to manually load audio streams from one or more audio file extensions. We are, however, going to make use of a library for audio management called **irrKlang**.

## Irrklang

![](0.png)

IrrKlang is a high level 2D and 3D cross platform \(Windows, Mac OS X, Linux\) sound engine and audio library that plays WAV, MP3, OGG, and FLAC files to name a few. It also features several audio effects like reverb, delay, and distortion that can be extensively tweaked.

> 3D audio means that an audio source can have a 3D position that will attenuate its volume based on the camera's distance to the audio source, making it feel natural in a 3D world \(think of gunfire in a 3D world; most often you'll be able to hear where it came from just by the direction/location of the sound\).

IrrKlang is an easy-to-use audio library that can play most audio files with just a few lines of code, making it a perfect candidate for our Breakout game. Note that irrKlang has a slightly restrictive license: you are allowed to use irrKlang as you see fit for non-commercial purposes, but you have to pay for their pro version whenever you want to use irrKlang commercially.

You can download irrKlang from their [download](http://www.ambiera.com/irrklang/downloads.html) page; we're using version 1.5 for this chapter. Because irrKlang is closed-source, we cannot compile the library ourselves so we'll have to do with whatever irrKlang provided for us. Luckily they have plenty of precompiled library files.

Once you include the header files of irrKlang, add their \(64-bit\) library \(irrKlang.lib\) to the linker settings, and copy the dll file\(s\) to the appropriate locations \(usually the same location where the .exe resides\) we're set to go. Note that if you want to load MP3 files, you'll also have to include the ikpMP3.dll file.

### Adding music

Specifically for this game I created a small little audio track so the game feels a bit more alive. You can find the audio track [here](breakout.mp3) that we'll use as the game's background music. This track is what we'll play whenever the game starts and that continuously loops until the player closes the game. Feel free to replace it with your own tracks or use it in any way you like. 

Adding this to the Breakout game is extremely easy with the irrKlang library. We include the irrKlang header file, create an `irrKlang::ISoundEngine`, initialize it with *createIrrKlangDevice*, and then use the engine to load and play audio files:

```cpp
#include <irrklang/irrKlang.h>
using namespace irrklang;

ISoundEngine *SoundEngine = createIrrKlangDevice();
  
void Game::Init()
{
    [...]
    SoundEngine->play2D("audio/breakout.mp3", true);
}
```

Here we created a SoundEngine that we use for all audio-related code. Once we've initialized the sound engine, all we need to do to play audio is simply call its play2D function. Its first parameter is the filename, and the second parameter whether we want the file to loop \(play again once it's finished\).

And that is all there is to it! Running the game should now cause your speakers \(or headset\) to violently blast out sound waves.

### Adding sounds

We're not there yet, since music by itself is not enough to make the game as great as it could be. We want to play sounds whenever something interesting happens in the game, as extra feedback to the player. Like when we hit a brick, or when we activate a powerup. Below you can find all the sounds we're going to use \(courtesy of freesound.org\):

- [bleep.mp3](bleep.mp3): the sound for when the ball hit a non-solid block. 
- [solid.wav](solid.wav): the sound for when the ball hit a solid block. 
- [powerup.wav](powerup.wav): the sound for when we the player paddle collided with a powerup block. 
- [bleep.wav](bleep.wav): the sound for when we the ball bounces of the player paddle. 

 Wherever a collision occurs, we play the corresponding sound. I won't walk through each of the lines of code where this is supposed to happen, but simply list the updated game code [here](game.cpp). You should easily be able to add the sound effects at their appropriate locations.

Putting it all together gives us a game that feels a lot more complete. All together it looks \(and sounds\) like this: 

[audio.mp4](audio.mp4)

IrrKlang allows for much more fine-grained control of audio controls like advanced memory management, audio effects, or sound event callbacks. Check out their simple C++ [tutorials](http://www.ambiera.com/irrklang/tutorials.html) and try to experiment with its features. 
