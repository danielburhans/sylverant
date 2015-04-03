# Introduction #
The database for Sylverant (needed only for the login server and shipgate -- not for individual ships if they are to be run separately from the shipgate/login server) consists of several tables. This page details the current layout of the database and provides the CREATE TABLE statements needed to create each table. Please note, these are all subject to change, and some have already changed several times.

# Table account\_data #
The account\_data table contains information on registered users of the server. The create table used is as follows:

```
CREATE TABLE `account_data` (
  `account_id` int(11) NOT NULL auto_increment,
  `username` varchar(18) collate latin1_general_ci NOT NULL,
  `password` varchar(33) collate latin1_general_ci NOT NULL,
  `email` varchar(255) collate latin1_general_ci NOT NULL,
  `regtime` int(10) unsigned NOT NULL,
  `isbanned` tinyint(2) NOT NULL default '0',
  `teamid` int(11) NOT NULL default '-1',
  `privlevel` int(10) unsigned NOT NULL default '0',
  `dressflag` tinyint(1) NOT NULL default '0',
  PRIMARY KEY  (`account_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_general_ci;
```

# Table bans #
The bans table is used for storing information related to a ban. This can be either a guildcard-level or IP level ban.

```
CREATE TABLE `bans` (
  `ban_id` int(11) NOT NULL auto_increment,
  `reason` varchar(256) default NULL,
  `startdate` int(10) unsigned NOT NULL default '0',
  `enddate` int(10) unsigned NOT NULL default '4294967295',
  `setby` int(11) default NULL,
  PRIMARY KEY  (`ban_id`),
  KEY `setby` (`setby`),
  CONSTRAINT `bans_ibfk_1` FOREIGN KEY (`setby`) REFERENCES `account_data` (`account_id`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Trigger ban\_start #
This trigger sets the start time of the ban to the current time when the ban is put into place.

```
CREATE TRIGGER `ban_start` BEFORE INSERT ON `bans` FOR EACH ROW set new.startdate = UNIX_TIMESTAMP();
```

# Table blueburst\_blacklist #
This table is used to store all the people that have been added to clients' blocked senders list on Blue Burst.

```
CREATE TABLE `blueburst_blacklist` (
  `guildcard` int(11) NOT NULL,
  `blocked_gc` int(11) NOT NULL,
  `name` tinyblob NOT NULL,
  `team_name` tinyblob NOT NULL,
  `text` tinyblob NOT NULL,
  `language` tinyint(4) NOT NULL,
  `section_id` tinyint(4) NOT NULL,
  `class` tinyint(4) NOT NULL,
  PRIMARY KEY  (`guildcard`,`blocked_gc`),
  KEY `guildcard` (`guildcard`),
  KEY `friend_gc` (`blocked_gc`),
  CONSTRAINT `blueburst_blacklist_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `blueburst_blacklist_ibfk_2` FOREIGN KEY (`blocked_gc`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table blueburst\_clients #
This table is used for storing clients that are authorized to play on Blue Burst. It stores the username and password combinations that they must use to log in.

```
CREATE TABLE `blueburst_clients` (
  `account_id` int(11) NOT NULL,
  `guildcard` int(11) NOT NULL,
  `username` varchar(33) character set utf8 NOT NULL,
  `password` binary(32) NOT NULL,
  PRIMARY KEY  (`account_id`),
  KEY `guildcard` (`guildcard`),
  CONSTRAINT `blueburst_clients_ibfk_1` FOREIGN KEY (`account_id`) REFERENCES `account_data` (`account_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `blueburst_clients_ibfk_2` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table blueburst\_guildcards #
This table is used to store the guildcards that Blue Burst users have collected from others.

```
CREATE TABLE `blueburst_guildcards` (
  `guildcard` int(11) NOT NULL,
  `friend_gc` int(11) NOT NULL,
  `name` tinyblob NOT NULL,
  `team_name` tinyblob NOT NULL,
  `text` tinyblob NOT NULL,
  `language` tinyint(4) NOT NULL,
  `section_id` tinyint(4) NOT NULL,
  `class` tinyint(4) NOT NULL,
  `comment` tinyblob,
  `priority` smallint(6) NOT NULL default '-1',
  PRIMARY KEY  (`guildcard`,`friend_gc`),
  KEY `guildcard` (`guildcard`),
  KEY `friend_gc` (`friend_gc`),
  CONSTRAINT `blueburst_guildcards_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `blueburst_guildcards_ibfk_2` FOREIGN KEY (`friend_gc`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Trigger blueburst\_guildcard\_insert\_prio #
This trigger is used to set the priority field properly on an insert into the blueburst\_guildcards table.

```
DELIMITER $
CREATE TRIGGER `blueburst_guildcard_insert_prio`
  BEFORE INSERT ON `blueburst_guildcards` FOR EACH ROW
  BEGIN
    DECLARE maxprio INT;
    SET maxprio = (SELECT MAX(`priority`) FROM `blueburst_guildcards` 
                     WHERE guildcard = NEW.guildcard);
    IF maxprio IS NOT NULL THEN
      SET NEW.priority = maxprio + 1;
    ELSE
      SET NEW.priority = 0;
    END IF;
  END$
DELIMITER ;
```

# Stored Procedure blueburst\_guildcard\_delete #
This stored procedure is used to actually delete a guildcard from a Blue Burst user's list. It automatically fixes priority values so that they stay in order. This should be used instead of a DELETE for actually removing the guildcards from the table.

```
DELIMITER $
CREATE PROCEDURE `blueburst_guildcard_delete` (IN gc INT, IN frgc INT)
  BEGIN
    DECLARE prio INT;
    SET prio = (SELECT `priority` FROM `blueburst_guildcards`
                  WHERE `guildcard` = gc AND `friend_gc` = frgc);
    IF prio IS NOT NULL THEN
      DELETE FROM `blueburst_guildcards`
        WHERE `guildcard` = gc AND `friend_gc` = frgc;
      UPDATE `blueburst_guildcards` SET `priority` = `priority` - 1
        WHERE `guildcard` = gc AND `priority` > prio;
    END IF;
  END$
DELIMITER ;
```

# Stored Procedure blueburst\_guildcard\_sort #
This stored procedure swaps two guildcards in a Blue Burst user's list (for when they use the sort guildcards menu option).

```
DELIMITER $
CREATE PROCEDURE `blueburst_guildcard_sort`
 (IN gc INT, IN frgc1 INT, IN frgc2 INT)
  BEGIN
    DECLARE prio1 INT;
    DECLARE prio2 INT;
    SET prio1 = (SELECT `priority` FROM `blueburst_guildcards`
                   WHERE `guildcard` = gc AND `friend_gc` = frgc1);
    SET prio2 = (SELECT `priority` FROM `blueburst_guildcards`
                   WHERE `guildcard` = gc AND `friend_gc` = frgc2);
    IF prio1 IS NOT NULL AND prio2 IS NOT NULL THEN
      UPDATE `blueburst_guildcards` SET `priority` = prio2
        WHERE `guildcard` = gc AND `friend_gc` = frgc1;
      UPDATE `blueburst_guildcards` SET `priority` = prio1
        WHERE `guildcard` = gc AND `friend_gc` = frgc2;
    END IF;
  END$
DELIMITER ;
```

# Table blueburst\_options #
This table is used to store the various game options set by Blue Burst users. The options column here is of type (from libsylverant) sylverant\_bb\_db\_opts\_t (see sylverant/characters.h).

```
CREATE TABLE `blueburst_options` (
  `guildcard` int(11) NOT NULL,
  `options` blob NOT NULL,
  PRIMARY KEY  (`guildcard`),
  CONSTRAINT `blueburst_options_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table character\_backup #
The character\_backup table is used to store automatic backups of character data. This functionality was added in shipgate protocol v11.

```
CREATE TABLE `character_backup` (
  `guildcard` int(11) NOT NULL,
  `name` varchar(32) NOT NULL,
  `data` blob NOT NULL,
  `size` smallint(6) default NULL,
  `time` timestamp NOT NULL default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP,
  PRIMARY KEY  (`guildcard`,`name`),
  CONSTRAINT `character_backup_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

# Table character\_data #
The character\_data table is used to store character data that is saved with the /save command on the server (type v1\_player\_t). It is also used to store Blue Burst character data as well (type sylverant\_bb\_db\_char\_t). Blue Burst character data is stored with slot values of 0 - 3.

```
CREATE TABLE `character_data` (
  `guildcard` int(11) NOT NULL,
  `slot` tinyint(4) NOT NULL,
  `data` blob NOT NULL,
  `size` smallint(6) default NULL,
  KEY `guildcard` (`guildcard`),
  CONSTRAINT `character_data_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_general_ci;
```

# Table dreamcast\_clients #
This table is obviously used to store Dreamcast clients for the server.

```
CREATE TABLE `dreamcast_clients` (
  `guildcard` int(11) NOT NULL,
  `serial_number` char(8) NOT NULL,
  `access_key` char(8) NOT NULL,
  `dc_id` char(8) NULL,
  PRIMARY KEY  (`guildcard`),
  CONSTRAINT `dreamcast_clients_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table dreamcast\_nte\_clients #
This table is used to store the serials/access keys of anyone on the Dreamcast Network Trial Edition.

```
CREATE TABLE `dreamcast_nte_clients` (
  `guildcard` int(11) NOT NULL,
  `serial_number` char(16) NOT NULL,
  `access_key` char(16) NOT NULL,
  `dc_id` char(8) NULL,
  PRIMARY KEY  (`guildcard`),
  CONSTRAINT `dreamcast_nte_clients_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table friendlist #
This table is used for storing friend lists of clients.

```
CREATE TABLE `friendlist` (
  `owner` int(11) NOT NULL,
  `friend` int(11) NOT NULL,
  `nickname` varchar(32) default NULL,
  KEY `owner` (`owner`),
  KEY `friend` (`friend`),
  CONSTRAINT `friendlist_ibfk_1` FOREIGN KEY (`owner`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `friendlist_ibfk_2` FOREIGN KEY (`friend`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table gamecube\_clients #
This table is used to store the serial number and access keys for Gamecube clients on the server.

```
CREATE TABLE `gamecube_clients` (
  `guildcard` int(11) NOT NULL,
  `serial_number` char(8) NOT NULL,
  `access_key` char(12) NOT NULL,
  PRIMARY KEY  (`guildcard`),
  CONSTRAINT `gamecube_clients_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table guildcard\_bans #
This table is used to set bans on a guildcard at the shipgate level.

```
CREATE TABLE `guildcard_bans` (
  `ban_id` int(11) NOT NULL,
  `guildcard` int(11) NOT NULL,
  PRIMARY KEY  (`ban_id`),
  KEY `guildcard` (`guildcard`),
  CONSTRAINT `guildcard_bans_ibfk_1` FOREIGN KEY (`ban_id`) REFERENCES `bans` (`ban_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `guildcard_bans_ibfk_2` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table guildcards #
This table is used to link accounts to their guildcards.

```
CREATE TABLE `guildcards` (
  `guildcard` int(11) NOT NULL auto_increment,
  `account_id` int(11) default NULL,
  PRIMARY KEY  (`guildcard`),
  KEY `account_id` (`account_id`),
  CONSTRAINT `guildcards_ibfk_1` FOREIGN KEY (`account_id`) REFERENCES `account_data` (`account_id`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=10000 DEFAULT CHARSET=latin1;
```

# Table ip\_bans #
This table is used to set a ban on an IP address at the shipgate level.

```
CREATE TABLE `ip_bans` (
  `ban_id` int(11) NOT NULL,
  `addr` int(10) unsigned NOT NULL,
  `mask` int(10) unsigned NOT NULL DEFAULT '4294967295',
  PRIMARY KEY (`ban_id`),
  CONSTRAINT `ip_bans_ibfk_1` FOREIGN KEY (`ban_id`) REFERENCES `bans` (`ban_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table online\_clients #
This table is used to store a list of currently online clients.

```
CREATE TABLE `online_clients` (
  `guildcard` int(11) NOT NULL,
  `name` varchar(32) NOT NULL,
  `ship_id` int(10) unsigned NOT NULL,
  `block` int(10) unsigned NOT NULL,
  `lobby` varchar(32) NOT NULL,
  `lobby_id` int(10) unsigned default NULL,
  `dlobby_id` int(10) unsigned default NULL,
  PRIMARY KEY  (`guildcard`),
  KEY `ship_id` (`ship_id`),
  KEY `lobby_id` (`lobby_id`),
  CONSTRAINT `online_clients_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `online_clients_ibfk_2` FOREIGN KEY (`ship_id`) REFERENCES `online_ships` (`ship_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

# Table online\_ships #
This one is actually used, and contains the list of currently online ships.

```
CREATE TABLE `online_ships` (
  `name` varchar(12) collate latin1_general_ci NOT NULL,
  `players` int(11) NOT NULL default '0',
  `ip` int(10) unsigned NOT NULL,
  `port` smallint(5) unsigned NOT NULL,
  `int_ip` int(10) unsigned NOT NULL,
  `ship_id` int(10) unsigned NOT NULL,
  `gm_only` tinyint(1) NOT NULL default '0',
  `games` int(11) NOT NULL default '0',
  `menu_code` int(11) NOT NULL default '0',
  `flags` int(10) unsigned NOT NULL default '0',
  `ship_number` int(10) unsigned NOT NULL default '0',
  `ship_ip6_high` bigint(20) unsigned default NULL,
  `ship_ip6_low` bigint(20) unsigned default NULL,
  `protocol_ver` int(10) unsigned NOT NULL,
  PRIMARY KEY  (`ship_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_general_ci;
```

# Table passwd\_resets #
This table is used by the website to manage requests for a password reset.

```
CREATE TABLE `passwd_resets` (
  `account_id` int(11) NOT NULL,
  `uuid` char(36) NOT NULL,
  `req_time` timestamp NOT NULL default CURRENT_TIMESTAMP,
  PRIMARY KEY  (`account_id`),
  CONSTRAINT `passwd_resets_ibfk_1` FOREIGN KEY (`account_id`) REFERENCES `account_data` (`account_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table pc\_clients #
This table is used to store the serial number and access key for PC clients.

```
CREATE TABLE `pc_clients` (
  `guildcard` int(11) NOT NULL,
  `serial_number` char(8) NOT NULL,
  `access_key` char(8) NOT NULL,
  PRIMARY KEY  (`guildcard`),
  CONSTRAINT `pc_clients_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# Table ship\_data #
This table contains the encryption keys of the ships allowed to connect to the shipgate. The main\_menu field in here controls whether that key is allowed in the main list of ships (a blank menu code).

```
CREATE TABLE `ship_data` (
  `idx` int(11) NOT NULL auto_increment,
  `main_menu` tinyint(3) unsigned NOT NULL default '0',
  `ship_number` int(10) unsigned NOT NULL,
  `memo` varchar(256) collate latin1_general_ci default NULL,
  `sha1_fingerprint` binary(20) NOT NULL,
  PRIMARY KEY  (`idx`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_general_ci;
```

# Table simple\_mail #
This table contains any simple mail sent to account holders while they are offline. When they log in, they'll get a message letting them know that they have mail to read on the website.

```
CREATE TABLE `simple_mail` (
  `message_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `recipient` int(11) NOT NULL,
  `status` tinyint(4) NOT NULL DEFAULT '0',
  `sender` int(11) DEFAULT NULL,
  `sender_name` varchar(32) NOT NULL,
  `sent_time` int(10) unsigned NOT NULL,
  `message` text NOT NULL,
  PRIMARY KEY (`message_id`),
  KEY `recipient` (`recipient`),
  KEY `sender` (`sender`),
  CONSTRAINT `simple_mail_ibfk_1` FOREIGN KEY (`recipient`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `simple_mail_ibfk_2` FOREIGN KEY (`sender`) REFERENCES `guildcards` (`guildcard`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

# Table user\_options #
This table contains options that have been set by users. Each option has its own format, so a tinyblob is used to make sure we can store anything (up to 255 bytes, anyway).

```
CREATE TABLE `user_options` (
  `guildcard` int(11) NOT NULL,
  `opt` int(10) unsigned NOT NULL,
  `value` tinyblob,
  PRIMARY KEY  (`guildcard`,`opt`),
  CONSTRAINT `user_options_ibfk_1` FOREIGN KEY (`guildcard`) REFERENCES `guildcards` (`guildcard`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```