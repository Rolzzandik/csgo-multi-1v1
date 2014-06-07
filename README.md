csgo-multi-1v1
=======================================
This is home of my work-in-progress CS:GO multi-1v1 arena plugin. It sets up any number of players in 1v1-situations on specially made maps and they fight in a ladder-type system. The winners move up, the losers go down.

Until the plugin version is greater than 1.0.0 you should consider this **unstable beta software**.

Also see the [AlliedModders thread](https://forums.alliedmods.net/showthread.php?t=241056).

### Features
- Round types: there are 3 round types: rifle, pistol, and awp
- Player selection: players can select to allow pistol and awp rounds or ban them, rifle rounds are always allowed
- Player preference: players can also select a preference of round type, if player preferences match they will play that type
- Weapon selection: players can select their primary (i.e. their rifle) and their pistol
- Armor on pistol rounds: helmets are taken away, and kevlar is also taken away if the player selected an upgraded pistol
- Optional flashbangs: players can select to "allow flashbangs" - if both players allow them, they each get 1
- ELO ranking system: optionally, player statistics can be stored in a database, see below for details



### Download
Download link: https://github.com/splewis/csgo-multi-1v1/releases

I **strongly** recommend using the updater plugin which can automatically update the plugin for bug fixes. It can be found at https://forums.alliedmods.net/showthread.php?t=169095.



### Donations
The easiest way to support my open development of this is to donate. You can do this at http://csgo1v1.splewis.net/donate, which will also give you a reserved slot for my servers.



### Installation
If you only want the plugin, either download **multi1v1.zip** or build it yourself.
It should contain the plugin binary (**plugins/multi1v1.smx**) and the default game config (**cfg/sourcemod/multi1v1/game_cvars.cfg**).
Extract these to the appropriate folders, tweak mul1v1.cfg if you want. The file **cfg/sourcemod/multi1v1/multi1v1.cfg** will be autogenerated when the plugin is first run and you can tweak it if you wish.

### Extras
My server has some extras such as the web-page for the stats and the !rank/!stats commands that show a player's web page with stats on it.
These are not included with the plugin but are not hard to implement yourself.

Additionally, while it would be possible to define the round types and available weapons in a easy-to-change config file, it would be rather difficult to match
the full level of flexibility modifying the code gives. In particular, a config that could remove kevlar on upgraded pistols would be non-trivial to implement.

If you want to write something custom in I can help, but I probably won't for free.



### Building
The build process is managed by the Makefile.

		make          # builds the .smx file
		make clean    # clears .smx files, .zip files
		make package  # packages the files to multi1v1.zip

To compile, you will need:
- [SMLib](http://www.sourcemodplugins.org/smlib/) (required)
- [Updater](https://forums.alliedmods.net/showthread.php?t=169095) (optional)


### Maps
I have a [workshop collection](http://steamcommunity.com/sharedfiles/filedetails/?id=249376192) of maps I use. Not all of the maps are finished though, so don't blindly use them all.

Guidelines for making a multi-1v1 map:
- Create 1 arena and test it well, and when are you happy copy it
- Create at least 9 arenas, I'd recommend at least 12, however. Any more than 16 is probably overkill.
- The players shouldn't be able to see each other on spawn
- Each arena should have exactly 2 spawns - one for CT's and one for T's (this is a condition that may be relaxed in the future)
- If you want to edit your map, it's easiest to delete all but 1 arena and re-copy them. Be warned this can cause issues with the game's lighting and clients may crash the first time they load the new map if they had downloaded the old one previously
- You should avoid areas where it's easy for 1 player to hide; ideally they should have to cover multiple angles if they sit in one spot



### Using the statistics database
You should add a database named mult1v1 to your databases.cfg file like so:

	"multi1v1"
	{
		"driver"			"mysql"
		"host"				"123.123.123.123"	// localhost works too
		"database"			"game_servers_database"
		"user"				"mymulti1v1server"
		"pass"				"strongpassword"
		"timeout"			"10"
		"port"			"3306"	// whatever port MySQL is set up on, 3306 is default
	}

To create a MySQL user and database on the database server, you can run:

		CREATE DATABASE game_servers_database;
		CREATE USER 'mymulti1v1server'@'123.123.123.123' IDENTIFIED BY 'strongpassword';
		GRANT ALL PRIVILEGES ON game_servers_database.multi1v1_stats TO 'mymulti1v1server'@'123.123.123.123';
		FLUSH PRIVILEGES;

Make sure to change the IP, the username, and the password. You should probably change the database as well, especially if you already have one set up you can use.

Schema:

		mysql> describe multi1v1_stats;
		+--------------+-------------+------+-----+---------+-------+
		| Field        | Type        | Null | Key | Default | Extra |
		+--------------+-------------+------+-----+---------+-------+
		| accountID    | int(11)     | NO   | PRI | 0       |       |
		| auth         | varchar(64) | NO   |     |         |       |
		| name         | varchar(64) | NO   |     |         |       |
		| wins         | int(11)     | NO   |     | 0       |       |
		| losses       | int(11)     | NO   |     | 0       |       |
		| rating       | float       | NO   |     | 1500    |       |
		+--------------+-------------+------+-----+---------+-------+


Note that the accountID field is what is returned by [GetSteamAccountID](https://wiki.alliedmods.net/SourceMod_1.5.0_API_Changes#Clients), which is "the lower 32 bits of the full 64-bit Steam ID (referred to as community id by some) and is unique per account."

The auth is the steam ID auth string.



### Clientprefs Usage/Cookies
Player choices (round type preferences, weapon choices) can be saved so they persist across maps for players (via the SourceMod clientprefs API).
Installing sqlite should be sufficient for this.

The reason the prefs/choice are not stored in the MySQL stats database, is because the data from it is not fetched immediately due to it taking longer.

### Other Notes
If you use an afk management plugin you may want to disable kicking spectators, because players are placed in a waiting queue on spectator when all the arenas are full.

