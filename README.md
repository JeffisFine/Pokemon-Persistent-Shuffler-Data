# Pokemon-Persistent-Shuffler-Data
Plugin for Bizhawk Shuffler 2 that persists specified data between shuffled Pokemon ROMs.

# Introduction
Persists specified data between shuffled games. Certain limitations apply, particularly between different gens. Those will be specified in each setting's name. This is just a project I picked up for fun on the side to fill time and learn Lua scripting. I'll happily take feedback on the Github, but don't expect professional support. Never used GitHub before this, hence the written changelog in this Readme. So cut me some slack. I'm not very imaginative, so I have no clue how useful this Shuffler plugin will actually be in a linear, turn-based RPG, but perhaps you can find some interesting challenges or whatever to do with it. See bottom of file for nerdy tech stuff.

# Requirements
- Potato PC or greater
- Bizhawk Emulator
- Bizhawk Shuffler 2
- Pokemon ROMs (initial support for Gens 1-3)

# Installation
1) Download Bizhawk Emulator and Pokemon Roms.
2) Install Bizhawk Shuffler 2 as directed by https://github.com/authorblues/bizhawk-shuffler-2.
3) Stage ROMs you will be using in the shuffler per Readme in Bizhawk Shuffler 2.
3) Download pokemon-persistent-data.zip.
4) Unzip into [Bizhwak Shuffler Home]\plugins.

# Setup
1) Confirm sha1 checksum of ROM is listed in [Bizhwak Shuffler Home]\plugins\pokemon-hashes.dat.
	- If hash not found, either download ROM with a sha1 that does match or follow the remaining 1x) steps to add entry to plugin.
	- If adding a new ROM, enter the following information in the pokemon-hashes.dat file in below format:
		```
		[SHA1 Hash] [tag] -- [File Name]
		ie. D7037C83E1AE5B39BDE3C30787637BA1D4C48CE2 pkmnrdbleng -- Pokemon - Blue Version (USA, Europe) (SGB Enhanced).gb
		```
		- Google operating system specific hashing function if you don't know it for your OS
		- [tag] is a designator lumping together ROMS with identical (or identical enough) RAM Addresses for key information. 
			- If you are using a ROM hack that does not alter the RAM Addresses for key information, then you can an already used tag. For instance, if you are using a FireRed English v1.1 ROM hack that just adds information in unused addresses, then [tag] can be pkmnfrlgeng. Otherwise, feel free to make up your own tag.
	- If you had to create a new [tag] in step b, add all relevant memory addresses in pokemon-persistent-data.lua, following the format of whichever gen the hack is most closely modeled after. This is the hard part! You'll need to add all relevant metadata for the settings you will be using. For instance, if you are running a Gen 1 ROM hack, you should know all the following addresses:
		```
		gen = 1
		,inbattle_addr = x
		,party_addr = 0xD163 -- Party count is first byte
		,party_count_addr = party_addr
		,party_bytes = 404
		,pokedex_addr = 0xD2F7
		,pokedex_bytes = 38
		,item_pocket_addr = 0xD31D
		,item_pocket_bytes = 42
		,money_addr = 0xD347
		,money_bytes = 3
		,pc_item_addr = 0xD53A
		,pc_item_bytes = 102
		,flags_addr = 0xD5A6
		,flags_bytes = 698
		,pc_pokemon_addr = 0xDA80
		,pc_pokemon_bytes = 1122
		```
2) Create shortcut to [Bizhawk Shuffler Home]\shuffle.lua in [Bizhawk Home]\Lua folder. (Not neccessary but fewer clicks)
3) Hard part's over; easy part now! Boot up Bizhawk.
4) Click Tools -> Lua Console.
5) Click Open Script (folder icon). This is different than Open Session!
6) Double-click shortcut of shuffle.lua
7) If starting a new session, uncheck "Resuming a Session?"
	- If resuming a session, just click Resume Previous Session
8) If "Persistent Pokemon Shuffler" isn't listed across from "Setup Plugins" or you wish to change settings from your last session, click "Setup Plugin"
	- Click dropdown list and choose Persistent Pokemon Shuffler Data
	- Check Enabled checkbox
	- Check settings you wish to use; uncheck settings you don'tag
	- Click Save and Close
9) Click Start New Session

# Data Structures
## Data Table: Primary structure passed between functions. Leverages built-in persistency of Bizhawk Shuffler.
	
	{
		tags[hash]		-- Table of tags based on sha1 hash of ROM. Tags group together ROMs with identical (enough) RAM Addresses.
		,prevtag		-- Tag of previously shuffled ROM
		,currtag		-- Tag of currently playing ROM
		,[tag] = {		-- Table of metadata and raw values of each tag needed for settings 
			metadata	-- RAM addresses, offsets, and byte sizes of necessary information
			,prevvalues	-- Raw strings, integers, and arrays saved from last time ROM with this tag was ran
		}
	}
	
## Data[tag].metadata: Exact fields in this table will depend on generation of ROM. Gen 3 example below:
	
	["pkmnfrlgeng"]={ -- FireRed/LeafGreen Version (USA, Europe) (Rev 1).gba
		gen = 3
		,inbattle_addr = 0x03003529
		,party_count_addr = 0x02024029
		,party_addr = 0x02024284
		,party_bytes = 600
		
		,save_block_1_pntr = 0x03005008
		,seen1_offset = 0x05F8
		,seen2_offset = 0x3A18
		,money_offset = 0x0290
		,money_bytes = 4
		,pc_item_offset = 0x0298
		,pc_item_bytes = 120
		,item_pocket_offset = 0x0310
		,item_pocket_bytes = 744
		,flags_offset = 0x0EE0
		,flags_bytes = 800
		
		,save_block_2_pntr = 0x0300500C
		,xor_key_offset = 0x0F20
		,pokedex_offset = 0x018
		,pokedex_bytes = 120
		,pokedex_seen_offset = 0x44
		,pokedex_seen_bytes = 52
		
		,save_block_3_pntr = 0x03005010
		,pc_pokemon_bytes = 33744
	}
	
## Data[tag].prevvalues: Stores information from previous ROM to be loaded into the RAM of current ROM upon loading
	
	{
	   ,party_count
	   ,party_values
	   ,pc_pokemon_values
	   ,pokedex_values
	   ,pokedex_seen_values
	   ,money_decoded_value
	   ,pc_items
	   ,pocket_items
	   ,flags_values
	}
	
# Change Log
- 0.8.0 - 3/5/23:
	- Streamlined data structure (can I stopped fiddling with it now, please?!)
	- Set stop-gap functions to prevent cross-gen ROM corruption from certain data transfers (ie. Pokemon)
- 0.7.2 - 3/4/23:
	- Fixed various edge-case bugs
- 0.7.1 - 2/24/23: 
	- Fixed syntax bugs introduced from overhaul. Passed DP-300 so I can stop bothering with that now! 
	- Need to fix logic error in on_frame if statement to check if values need to be set in early game scenarioes.
- 0.7.0 - 2/20/23:
	- Oveerhauled how data is passed between games
- 0.6.0 - 2/15/23:
	- Finished RAM addresses for the rest of Gen 3 English. Corrected a few errors in byte length for some segments.
- 0.5.0 - 2/5/2023:
	- Added RAM addresses for Gen 1 & 2 English games. The remaining Gen 3 games will need more research.
- 0.4.0 - 1/28/2023:
	- Cleaned up code and implemented using the "data" structure native to BS2 where appropriate
	- Create framework for possible cross-gen shuffling
- 0.3.0 - 1/23/2023:
	- Added contengencies so pushed RAM states are preserved on the first run of a ROM.
	- Added option to preserve RAM state if a new game is made after the first one for some reason.
- 0.2.0 - 1/22/2023:
	- Added Setting to not swap during battles. Provides more stability as stats and pokemon aren't changing in the middle of battle. Not game breaking if setting not used, though.
- 0.1.0 - 1/21/2023: 
	- Updates Pokemon RAM information upon shuffle to new game. Can currently sync the following:
		-Party
		-PC Boxes (Slow; need to improve)
		-Pokedex
		-Money
		-Items
		-Events (will probably lead to softlocks eventually)
	- Currently set up to work with FireRed/LeafGreen. Will expand in future.
		
# Future Changes
- Speed up PC Pokemon transfer rate
- Add additional gens (requires RAM Maps and/or decompilations)
- Allow cross-gen transfer of pokemon within ROMs' physical limitations
- Atomize the larger scale data groups (ie. give settings to transfer event flags but not beaten trainer flags)
- Whatever good ideas I hear from others
