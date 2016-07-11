---
layout: post
title: Messages Attachment Manager 
---
The [Messages Attachment Manager](https://github.com/connormurray7/message-attachment-manager) is a Mac application that allows you to select and delete the attachments that you have sent and received from people in the Messages application. This is only for people that use Messages (iMessage) on their Mac, if you haven't linked Messages up to your iCloud account then you won't have all of the files on your computer.

![alt text](/images/Message-Attachment-Manager.png "Title")

### _Credits_


I would like to give credit to Ray Wenderlich for the [SlidesMagic](https://www.raywenderlich.com/120494/collection-views-os-x-tutorial) tutorial that I used for the backbone of the UI of the app. In addition I used Stephen Celis' [SQLite.swift](https://github.com/stephencelis/SQLite.swift) project for SQL calls to the database. 

### _Why I made this_
Being one of those people that has a 128 GB Macbook Air, I have to be cognizant of what is on my computer and make sure to keep it relatively clean. There are all those stories of people with humongous "Other" storage (currently I have 70 GB of "Other"). I use Messages very liberally when it comes to sending various files, and I realized that I had about 20 GB of attachments saved at one point...

How is this possible? Well, when you send someone a picture on your phone, that picture gets stored on your phone, laptop, and any other devices that Messages is set up in. (Not to mention that picture also get stored on every device on the receiver's end). This poses a very difficult technical challenge for Apple that I won't try to solve, but what I did do was make it possible to mass-delete some of these attachments on your Mac.

One of the biggest annoyances that I have had with Messages on both the Mac and iOS platform was that there is no good way to clear out the pictures and other attachments that you send other people. You can clear multiple attachments at once by by going to "details" and then selecting multiple attachments to delete. But you have to do this for every individual person or group conversation that you have. 

So I thought, why not just hack together something that can do the job visually. 


### _How Messages are Stored on the Mac_

This might be useful for any developers that want to interact with the Messages application. 

All of the important information for the Messages application are stored in `~/Library/Messages/`. There there are 3 files,
    
- `chat.db` - stores the message data and meta-data for each message
- `chat.db-wal` - temporary file for SQLite3
- `chat.db-shm` - temporary file for SQLite3

`chat.db` is where all of the important information that is stored. The table schemas are as follows

	CREATE TABLE _SqliteDatabaseProperties 
		(key TEXT, value TEXT, UNIQUE(key));
	
	CREATE TABLE chat (
	ROWID INTEGER PRIMARY KEY AUTOINCREMENT, 
	guid TEXT UNIQUE NOT NULL, style INTEGER, 
	state INTEGER, account_id TEXT, 
	properties BLOB, 
	chat_identifier TEXT, 
	service_name TEXT, 
	room_name TEXT, 
	account_login TEXT, 
	is_archived INTEGER DEFAULT 0, 
	last_addressed_handle TEXT, 
	display_name TEXT, 
	group_id TEXT, 
	is_filtered INTEGER DEFAULT 0, 
	successful_query INTEGER DEFAULT 1
	);

	CREATE TABLE attachment (
	ROWID INTEGER PRIMARY KEY AUTOINCREMENT, 
	guid TEXT UNIQUE NOT NULL, 
	created_date INTEGER DEFAULT 0, 
	start_date INTEGER DEFAULT 0, 
	filename TEXT, 
	uti TEXT, 
	mime_type TEXT, 
	transfer_state INTEGER DEFAULT 0, 
	is_outgoing INTEGER DEFAULT 0, 
	user_info BLOB, 
	transfer_name TEXT, 
	total_bytes INTEGER DEFAULT 0
	);
	
	CREATE TABLE handle ( 
	ROWID INTEGER PRIMARY KEY AUTOINCREMENT UNIQUE, 
	id TEXT NOT NULL, 
	country TEXT, 
	service TEXT NOT NULL, 
	uncanonicalized_id TEXT, 
	UNIQUE (id, service) 
	);

	CREATE TABLE chat_handle_join ( 
	chat_id INTEGER REFERENCES chat (ROWID) ON DELETE CASCADE, 
	handle_id INTEGER REFERENCES handle (ROWID) ON DELETE CASCADE, 
	UNIQUE(chat_id, handle_id)
	);

	CREATE TABLE message (
	ROWID INTEGER PRIMARY KEY AUTOINCREMENT, 
	guid TEXT UNIQUE NOT NULL, 
	text TEXT, 
	replace INTEGER DEFAULT 0, 
	service_center TEXT, 
	handle_id INTEGER DEFAULT 0, 
	subject TEXT, 
	country TEXT, 
	attributedBody BLOB, 
	version INTEGER DEFAULT 0, 
	type INTEGER DEFAULT 0, 
	service TEXT, 
	account TEXT, 
	account_guid TEXT, 
	error INTEGER DEFAULT 0, 
	date INTEGER, 
	date_read INTEGER, 
	date_delivered INTEGER, 
	is_delivered INTEGER DEFAULT 0, 
	is_finished INTEGER DEFAULT 0, 
	is_emote INTEGER DEFAULT 0, 
	is_from_me INTEGER DEFAULT 0, 
	is_empty INTEGER DEFAULT 0, 
	is_delayed INTEGER DEFAULT 0, 
	is_auto_reply INTEGER DEFAULT 0, 
	is_prepared INTEGER DEFAULT 0, 
	is_read INTEGER DEFAULT 0, 
	is_system_message INTEGER DEFAULT 0, 
	is_sent INTEGER DEFAULT 0, 
	has_dd_results INTEGER DEFAULT 0, 
	is_service_message INTEGER DEFAULT 0, 
	is_forward INTEGER DEFAULT 0, 
	was_downgraded INTEGER DEFAULT 0, 
	is_archive INTEGER DEFAULT 0, 
	cache_has_attachments INTEGER DEFAULT 0, 
	cache_roomnames TEXT, 
	was_data_detected INTEGER DEFAULT 0, 
	was_deduplicated INTEGER DEFAULT 0, 
	is_audio_message INTEGER DEFAULT 0, 
	is_played INTEGER DEFAULT 0, 
	date_played INTEGER, 
	item_type INTEGER DEFAULT 0, 
	other_handle INTEGER DEFAULT -1, 
	group_title TEXT, 
	group_action_type INTEGER DEFAULT 0, 
	share_status INTEGER, 
	share_direction INTEGER, 
	is_expirable INTEGER DEFAULT 0, 
	expire_state INTEGER DEFAULT 0, 
	message_action_type INTEGER DEFAULT 0, 
	message_source INTEGER DEFAULT 0
	);

	CREATE TABLE chat_message_join ( 
	chat_id INTEGER REFERENCES chat (ROWID) ON DELETE CASCADE, 
	message_id INTEGER REFERENCES message (ROWID) ON DELETE CASCADE, 
	PRIMARY KEY (chat_id, message_id)
	);

	CREATE TABLE message_attachment_join ( 
	message_id INTEGER REFERENCES message (ROWID) ON DELETE CASCADE, 
	attachment_id INTEGER REFERENCES attachment (ROWID) ON DELETE CASCADE, 
	UNIQUE(message_id, attachment_id)
	);
	
 Notice that because there is a message_attachment_join, not only does the attachment reference need to be deleted from the attachment table, but also from the join table. 
 
 The final item is the `Attachments/` directory. Within `Attachments/` there are exactly `256` folders ranging from `00` to `ff`. For those that are unfamiliar or forgot, there are 8 bits in a byte, so there are zero to $$2^8 = 256$$ bit permutations in a byte. 
 
 **If** the directory contains attachments **then** the directory will contain more directories each of which would be from `00-ff`. Finally within the final level there will be  directories named after the attachments GUID (globally unique identifier). For example one of my attachments' path is
 
 
	/Users/<user_name>/Library/Messages/Attachments/0c/12/27F70B0F-0C46-4FD4-BF22-85E9D58DEABC/img.png
 	


### _User defined functions (UDF) and SQLite_

The biggest hurdle with this database is that there are some user defined functions that were created by Apple in Obj-C or C. Because Messages is not open-source we do not know how those functions are defined, or what they do, and we are just left with the references that are left in the database. For example,

	CREATE TRIGGER before_delete_on_attachment BEFORE DELETE ON attachment BEGIN   SELECT before_delete_attachment_path(OLD.ROWID, OLD.guid); END;

	CREATE TRIGGER after_delete_on_attachment AFTER DELETE ON attachment BEGIN   SELECT delete_attachment_path(OLD.filename); END;

These two triggers contain functions **before_delete_attachment_path** and **delete_attachment_path** which cause problems because the functions are not stored in the database, so if you load it, your instance won't recognize the functions and fail.

After a lot of poking around (definitely broke my chat.db a couple of times) the best work-around that I found was to drop the triggers and re-add them when deleting rows. This is probably the least maintainable way to do it, but unfortunately SQLite3 also does not have the "ALTER" trigger keyword so we couldn't even disable and re-enable them.

If anyone has a more clever way around the UDF constraint, please let me know.

