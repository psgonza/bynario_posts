title: Overwriting Windows keyboard shortcuts with AHK
date: 2012-04-17 14:06:00+02:00
slug: 2012-04-17-overwriting-windows-keyboard-shortcuts-with-ahk
category: posts
tag: scripts , AHK

I was looking for an easy way to put my screen on sleep mode when I locked my laptop using WINDOWS - L, and also to mute the music player (in my case, foobar2000 or spotify)

By using [AutoHotKeys](http://www.autohotkey.com/), you can tweak your Windows keyboard shortcuts and make the most of them.

This is how I made it:

```
;###########################################################################
; LOCK WINDOWS & STOP FOOBAR || SPOTIFY & POWER OFF SCREEN (WIN - L) 
;###########################################################################

#l::
Run,%A_WinDir%\System32\rundll32.exe user32.dll`,LockWorkStation
Sleep 1500
SendMessage 0x112, 0xF170, 2,,Program Manager
Process, Exist, foobar2000.exe
If (ErrorLevel <> 0) ; If it is not running
{
Run "C:\Program Files\foobar2000\foobar2000.exe" /stop
}
Process, Exist, spotify.exe

If (ErrorLevel <> 0) ; If it is not running
{
DetectHiddenWindows, On
WinGetTitle, spotifyTitle, ahk_class SpotifyMainWindow
spotifyTitleLength := StrLen(spotifyTitle)
spotifyActive := InStr(currentTitle, "Spotify") 
spotifyPlaying := spotifyTitleLength > 7
if spotifyPlaying {
ControlSend, ahk_parent, {Space}, ahk_class SpotifyMainWindow
}
DetectHiddenWindows, Off
}

return
```

Working like a charm for me ;)
