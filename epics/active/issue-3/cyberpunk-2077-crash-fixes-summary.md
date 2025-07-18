# Cyberpunk 2077 Crash Fixes Summary
## EXCEPTION_ACCESS_VIOLATION (0xC0000005) & Flatline Errors

### Research Summary
This document compiles 40+ potential fixes for Cyberpunk 2077 crashes on Windows 11, particularly the EXCEPTION_ACCESS_VIOLATION (0xC0000005) error that started after the April 2025 update. Fixes are prioritized from most commonly suggested to least common.

---

## TOP PRIORITY FIXES (Most Commonly Recommended)

### 1. **Disable Third-Party Software Conflicts**
- **Taskbar Organization Software (Bins)**: Most common culprit - disable or exclude Cyberpunk from Bins settings
- **Nahimic Audio Software**: Disable via services.msc or uninstall completely
- **Third-party Antivirus**: Temporarily disable (especially Avira Security)
- Sources: [GOG Forums](https://www.gog.com/forum/cyberpunk_2077/solved_exception_access_violation_0xc0000005_for_me), [Driver Easy](https://www.drivereasy.com/knowledge/solved-cyberpunk-2077-has-flatlined-error/)

### 2. **Update Graphics Drivers**
- Download latest NVIDIA/AMD drivers specifically optimized for Cyberpunk 2077
- NVIDIA users: Check for hotfix drivers (e.g., v576.66) released for game crashes
- Consider rolling back if crashes started after recent driver update
- Sources: [NVIDIA GeForce Forums](https://www.nvidia.com/en-us/geforce/forums/game-ready-drivers/13/556589/), [TechPowerUp](https://www.techpowerup.com/337641/)

### 3. **Verify Game Files**
- **Steam**: Right-click → Properties → Installed Files → Verify integrity
- **GOG**: Manage installation → Verify/Repair
- **Epic**: Similar verification option in launcher
- Sources: [CD Projekt Red Support](https://support.cdprojektred.com/en/cyberpunk/pc/sp-technical/issue/1700/my-game-crashes-7), [Steam Community](https://steamcommunity.com/app/1091500/discussions/7/3462731694143826580/)

### 4. **Remove Bluetooth Devices and Third-party Peripherals**
- Disconnect all Bluetooth devices
- Remove HOTAS, controllers, and other USB peripherals
- Test with only essential mouse/keyboard
- Sources: [RespawnFirst](https://respawnfirst.com/cyberpunk-2077-unhandled-exception-0xc0000005-fix/), [KeenGamer](https://www.keengamer.com/articles/guides/7-ways-to-fix-the-cyberpunk-2077-has-flatlined-error-on-a-windows-pc/)

### 5. **Disable In-Game Overlays**
- Steam overlay
- Discord overlay
- GeForce Experience overlay
- GOG Galaxy overlay
- Sources: [GameRevolution](https://www.gamerevolution.com/guides/669190-whoa-cyberpunk-2077-has-flatlined-error-fix-for-pc), [Appuals](https://appuals.com/how-to-fix-cyberpunk-2077-has-flatlined-error-on-windows-10/)

## HIGH PRIORITY FIXES

### 6. **File Permissions Fix**
- Navigate to: %UserProfile%\AppData\Local\CD Projekt Red
- Remove 'read-only' attribute from folder
- Check game shortcut properties for read-only setting
- Sources: [GOG Support](https://support.gog.com/hc/en-us/articles/4408556606237), [Reddit Community](https://steamcommunity.com/app/1091500/discussions/7/3370404731619570155/)

### 7. **Clear DirectX Shader Cache**
- Navigate to: C:\Users\[username]\AppData\Local\NVIDIA\DXCache
- Delete entire DXCache folder
- DirectX will rebuild cache automatically
- Sources: [Clawsome Gamer](https://clawsomegamer.com/cyberpunk-2077-unhandled-exception-0xc0000005-fix/), [Benisnous](https://benisnous.com/tag/fix-cyberpunk-2077-exception_access_violation-0xc0000005/)

### 8. **Run as Administrator + Compatibility Mode**
- Right-click Cyberpunk2077.exe → Properties
- Check "Run as administrator"
- Check "Disable fullscreen optimizations"
- Set compatibility mode to Windows 7
- Sources: [Driver Easy](https://www.drivereasy.com/knowledge/cyberpunk-2077-crashing-on-pc-solved/), [Prima Games](https://primagames.com/tips/how-to-fix-cyberpunk-2077-crashing-on-startup)

### 9. **Remove/Update Mods**
- Uninstall ALL mods and modding tools (REDMod, Cyber Engine Tweaks, etc.)
- Delete r6/cache folder
- Verify files after mod removal
- Sources: [RedModding Wiki](https://wiki.redmodding.org/cyberpunk-2077-modding/for-mod-users/user-guide-troubleshooting), [CD Projekt Red Forums](https://support.cdprojektred.com/en/cyberpunk/pc/sp-technical)

### 10. **System File Checker (SFC)**
- Open Command Prompt as administrator
- Run: sfc /scannow
- Let scan complete and repair issues
- Sources: [Windows Dispatch](https://www.windowsdispatch.com/fix-cyberpunk-2077-flatlined-crash-error/), [Microsoft Support](https://answers.microsoft.com/en-us/windows/forum/all/exceptionaccessviolation/841fa4d7-a1cf-49c1-979f-fffbefefb33b)

## MEDIUM PRIORITY FIXES

### 11. **Page File/Virtual Memory Configuration**
- Set custom page file size to 3x physical RAM or minimum 4GB
- Or use system-managed size
- Some users report disabling page file helps (risky)
- Sources: [KeenGamer](https://www.keengamer.com/articles/features/troubleshooting/7-ways-to-fix-random-in-game-cyberpunk-2077-crashing-on-windows-11-10-pcs/), [Steam Community](https://steamcommunity.com/app/1091500/discussions/0/3010060319385183660/)

### 12. **BIOS Settings (Intel 13th Gen)**
- Change SVID Behavior from "auto" to "Intel's fail safe"
- Adjust CPU Vcore to "Normal" + Dynamic Vcore (DVID) to +0.005V or +0.015V
- Disable XMP, MultiCore Enhancement, C States
- Sources: [Tom's Hardware](https://forums.tomshardware.com/threads/crashing-in-cyberpunk-2077.3838910/), [Intel Community](https://community.intel.com/t5/Processors/i9-13900K-very-frequent-crashes-Windows-11-with-apps-games-and/m-p/1527297)

### 13. **Disable CPU/GPU Overclocking**
- Turn off MSI Afterburner
- Reset to default clock speeds
- Factory overclocked GPUs may need underclocking
- Sources: [Game Pressure](https://www.gamepressure.com/newsroom/cyberpunk-2077-21-crashing-fix/z466dd), [XDA Developers](https://www.xda-developers.com/why-pc-crashes-mid-game/)

### 14. **Visual C++ Redistributables Repair**
- Uninstall all Visual C++ packages
- Download and install latest versions from Microsoft
- Use repair option if available
- Sources: [Game Errors](https://gameserrors.com/fixed-cyberpunk-2077-flatlined/), [MiniTool](https://www.partitionwizard.com/partitionmagic/cyberpunk-2077-has-flatlined.html)

### 15. **TDR (Timeout Detection Recovery) Registry Edit**
- Increase TDRDelay from default 2 seconds to 10 seconds
- Requires registry editing (advanced users)
- Sources: [Systweak](https://www.systweak.com/blogs/how-to-resolve-the-exception_access_violation-in-windows-11-10/), [Tenorshare](https://4ddig.tenorshare.com/windows-fix/exception-access-violation.html)

### 16. **Close Background Applications**
- Task Manager → End non-essential processes
- Disable startup programs
- Free up RAM for Cyberpunk
- Sources: [Clawsome Gamer](https://clawsomegamer.com/cyberpunk-2077-pc-crash-fix/), [Game Rant](https://gamerant.com/cyberpunk-2077-flatlined-steam-crash-guide/)

### 17. **Delete UserSettings.json**
- Navigate to: AppData\Local\CD Projekt Red\Cyberpunk 2077
- Backup and delete UserSettings.json
- Game will create new settings file
- Sources: [PCGamingWiki](https://www.pcgamingwiki.com/wiki/Cyberpunk_2077), [Igor's Lab](https://www.igorslab.de/en/cyberpunk-2077-new-features-are-great-but-what-to-do-if-patch-2-1-crashes-a-workaround/)

### 18. **Install No Intro Videos Mod**
- Unusual fix: Installing mod to skip intro videos
- Available on Nexus Mods
- Sources: [Nexus Mods Community](https://www.nexusmods.com/cyberpunk2077), [Steam Forums](https://steamcommunity.com/app/1091500/discussions/7/3394049806857960538/)

## LOWER PRIORITY FIXES

### 19. **Update Windows 11**
- Install latest Windows updates
- Some users report rolling back to Windows 10 helps
- Sources: [Tom's Hardware](https://forums.tomshardware.com/threads/constant-exception-access_violation-crashes-in-all-games-0xc0000005-after-fresh-windows-11-install.3784841/), [Windows Central](https://www.windowscentral.com/cyberpunk-2077-crashing-issues-and-how-fix-them)

### 20. **Revert to Previous Game Version**
- Steam: Properties → Betas → 1_63_legacy_patch
- Works until patches fix current issues
- Sources: [Steam Community](https://steamcommunity.com/app/1091500/discussions/0/4036977429237135001/), [eXputer](https://exputer.com/guides/errors/cyberpunk-2077-keeps-crashing/)

### 21. **Check Installation Path**
- Ensure no non-English characters in path
- Move installation if necessary
- Sources: [RedModding Wiki](https://wiki.redmodding.org/cyberpunk-2077-modding/for-mod-users/user-guide-troubleshooting), [GitHub CDPR Docs](https://github.com/CDPR-Modding-Documentation/Cyberpunk-Modding-Docs)

### 22. **Power Supply Check**
- Remove 24-pin ATX extension cables
- Connect PSU directly to motherboard
- Ensure adequate wattage for system
- Sources: [XDA Developers](https://www.xda-developers.com/why-pc-crashes-mid-game/), [ResetEra Forums](https://www.resetera.com/threads/changing-page-file-size-in-windows-10-please-halp.412062/)

### 23. **Disable Ray Tracing**
- In-game graphics settings
- Ray Tracing causes crashes for many users
- Sources: [MiniTool](https://www.partitionwizard.com/partitionmagic/cyberpunk-2077-crashing.html), [UGetFix](https://ugetfix.com/ask/fix-cyberpunk-2077-crashing-flatlined-ce-34878-0-error-and-low-fps/)

### 24. **AVX CPU Instruction Set Check**
- Old CPUs lacking AVX won't run game
- No fix except CPU upgrade
- Sources: [RespawnFirst](https://respawnfirst.com/cyberpunk-2077-flatlined-error-fix/), [MS Codes](https://ms.codes/blogs/computer-hardware/cyberpunk-2077-amd-cpu-fix)

### 25. **Event ID 10010 Fix**
- Check Event Viewer for ID 10010
- Related to DCOM permissions
- Sources: [Microsoft Learn](https://learn.microsoft.com/en-us/answers/questions/1807558/event-id-10010-causing-cyberpunk-2077-to-crash), [Windows Club](https://www.thewindowsclub.com/cyberpunk-2077-keeps-crashing)

### 26. **Clean GPU Driver Install (DDU)**
- Use Display Driver Uninstaller
- Boot to safe mode
- Complete clean installation
- Sources: [Driver Easy](https://www.drivereasy.com/knowledge/cyberpunk-2077-crashing-on-pc-solved/), [AMD Community](https://community.amd.com/t5/pc-graphics/crash-on-cyberpunk-2077/m-p/703271)

### 27. **Cross-Platform Save Fix**
- Ensure game is fully updated
- Open Load Game menu
- Press Cross Progression button
- Sources: [CD Projekt Red Support](https://support.cdprojektred.com/en/cyberpunk/pc/sp-technical/issue/2425/how-to-use-cross-progression-1), [The Droid Guy](https://thedroidguy.com/fix-cyberpunk-2077-crashing-issues-1145147)

### 28. **Temperature Monitoring**
- Use HWMonitor or Core Temp
- Check if crashes correlate with high temps
- Improve cooling if needed
- Sources: [Tom's Hardware](https://forums.tomshardware.com/threads/cyberpunk-crashing-or-gpu-is-dead.3836320/), [Linus Tech Tips](https://linustechtips.com/topic/1282816-cyberpunk-2077-amd-driver-crash/)

### 29. **REDMod Cache Clear**
- Delete r6/cache folder
- Delete r6/scripts contents
- Manually reinstall mods
- Sources: [Nexus Mods](https://www.nexusmods.com/cyberpunk2077/mods/1511), [GOG Support](https://support.gog.com/hc/en-us/articles/360018794397)

### 30. **Adjust Graphics Settings**
- Lower overall graphics quality
- Disable specific features causing crashes
- Start with lowest settings, increase gradually
- Sources: [Prima Games](https://primagames.com/tips/how-to-fix-cyberpunk-2077-crashing-on-startup), [Windows Central](https://www.windowscentral.com/software-apps/windows-11/how-to-manage-virtual-memory-on-windows-11)

### 31. **Check RAM for Hardware Issues**
- Use Windows Memory Diagnostic
- Or MemTest86
- Replace faulty RAM if detected
- Sources: [Overclock.net](https://www.overclock.net/threads/should-i-put-my-page-file-on-a-ram-disk.1802983/), [CD Projekt Red Forums](https://forums.cdprojektred.com/index.php?threads/cyberpunk-2077-pc-crash-issues.11040656/)

### 32. **Disable Hyperthreading (BIOS)**
- Access BIOS settings
- Disable Hyperthreading/SMT
- May improve stability
- Sources: [Intel Community](https://community.intel.com/t5/Processors/i9-13900K-very-frequent-crashes-Windows-11-with-apps-games-and/m-p/1534641), [Game Pressure](https://www.gamepressure.com/newsroom/2023-cyberpunk-2077-has-flatlined-20-crashing-explained/z9601b)

### 33. **Repair Game Installation**
- Through launcher repair option
- Or complete reinstall
- Keep base game files if possible
- Sources: [CD Projekt Red Support](https://support.cdprojektred.com/en/cyberpunk/pc/sp-technical), [eXputer](https://exputer.com/guides/errors/cyberpunk-2077-keeps-crashing/)

### 34. **Disable Windows Game Mode**
- Windows Settings → Gaming
- Turn off Game Mode
- Sources: [Appuals](https://appuals.com/fix-cyberpunk-2077-crashing-on-startup/), [Javelin Tech](https://www.javelin-tech.com/blog/2023/05/virtual-memory-solidworks-performance/)

### 35. **Check Discord Hooks**
- Discord integration can cause crashes
- Disable Discord game detection
- Sources: [RedModding Discord](https://discord.gg/redmodding), [Cyber Engine Tweaks Wiki](https://wiki.redmodding.org/cyber-engine-tweaks/getting-started/installing/troubleshooting)

### 36. **Windows Defender Exclusion**
- Add game folder to Windows Defender exclusions
- Prevents real-time scanning interference
- Sources: [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/1807558/), [CD Projekt Red Technical Support](https://support.cdprojektred.com/en/cyberpunk/pc/sp-technical)

### 37. **Update Chipset Drivers**
- Download from motherboard manufacturer
- Can resolve stability issues
- Sources: [Tom's Hardware](https://forums.tomshardware.com/threads/crashing-in-cyberpunk-2077.3838910/), [Steam Community](https://steamcommunity.com/app/1091500/discussions/7/4029096058474839686/)

### 38. **Disable Hardware Acceleration**
- In Discord, browsers, other apps
- Reduces GPU conflicts
- Sources: [XDA Developers](https://www.xda-developers.com/so-frustrated-nvidia-drivers/), [Steam Discussions](https://steamcommunity.com/app/1091500/discussions/0/3875968032469427104/)

### 39. **Check Event Viewer**
- Windows Event Viewer for crash details
- Look for specific error codes
- Sources: [Microsoft Support](https://answers.microsoft.com/en-us/windows/forum/all/games-keep-crashing-with-no-error-message-memory/779f641f-0666-4936-bc77-c30005d0e78c), [CD Projekt Red Forums](https://forums.cdprojektred.com/index.php?threads/cyberpunk-2077-pc-crash-issues.11040656/page-55)

### 40. **Factory Reset Windows (Last Resort)**
- Complete Windows reinstall
- Back up data first
- Sources: [Windows Dispatch](https://www.windowsdispatch.com/fix-cyberpunk-2077-flatlined-crash-error/), [The Windows Club](https://www.thewindowsclub.com/cyberpunk-2077-keeps-crashing)

---

## Additional Resources

- **CD Projekt Red Official Support**: https://support.cdprojektred.com/en/cyberpunk/pc/sp-technical
- **RedModding Discord**: https://discord.gg/redmodding
- **Steam Community Forums**: https://steamcommunity.com/app/1091500/discussions/
- **GOG Support**: https://support.gog.com/hc/en-us/sections/115000182829-Cyberpunk-2077

---

## Notes

- Fixes effectiveness varies by system configuration
- Try simpler fixes first before advanced solutions
- Always backup save files and settings before major changes
- Some fixes may conflict with each other
- Cross-platform saves require game to launch successfully first

**Compiled**: July 18, 2025  
**Total Fixes Found**: 40+