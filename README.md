# README #

**Overview**

We are going to create a simple chat app for andorid devices using Socket. In this app we have two Screens. The first Screen will prompt the user to enter his/her name. This name will display during chat on second screen. The second screen shows all the chat stuff and compose new message to send.This mss Chat app has ability to save your chat history into sqlite database, so you can see your history when you open your app second time. 



### Main Components use in this app:- ###
   

          1) Web Server socket url:- "ws://192.168.0.143:9000/server.php"; (You can add your own Web server socket url )
          2) Sqlite Database:- To save history. 
          3) Backup Store in database.
          4) Android websockets library:- For Establish connection with server and get response .
          5) Shared Prefrences:- To save user details .


### Installation Steps:-  ###

1) Download MChatApp Source code from :- [https://bitbucket.org/mssdineshsambial/mchatapp](https://bitbucket.org/mssdineshsambial/mchatapp)

2) Import MChatApp Project :-
         
             a) Right Click--->Import-->Existing Android code into workspace 
                           -->Browse(MChatApp Source Code)

3) Web Server Socket URL Add to WsConfig.java file:- 
              URL_WEBSOCKET = "ws://192.168.0.143:9000/server.php"; 	      
      (You can add your own Web server socket url here)
### Add permissions into AndroidManifest.xml file ###

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

**Clean your project and run.**

====================================================================================

###How this code works:-###

** 1. Creating web socket client. Connect with Web server socket **
	
	private void createWebSocketClient() {

		client = new WebSocketClient(URI.create(WsConfig.URL_WEBSOCKET), new WebSocketClient.Listener() {
			@Override
			public void onConnect() {
			}
			
			@Override
			public void onMessage(String message) {
				Log.d(TAG, String.format("Got string message! %s", message));
				System.out.println("Message Recieved: " + message);
				parseMessage(message);
			}
	
			@Override
			public void onDisconnect(int code, String reason) {
			}

			@Override
			public void onError(Exception error) {
				Log.e(TAG, "Error! : " + error);
				// showToast("Error! : " + error);
			}
		}, null);

		client.connect();
	}

** 2. On Send Button Click, Send message  to web server  **


	private void sendMessageToServer(String message) {
		if (client != null && client.isConnected()) {
			JSONObject jsonObj = new JSONObject();
			try {
				jsonObj.put("message", message);
				jsonObj.put("name", name);
				jsonObj.put("color", "#eb543b");
			} catch (JSONException e) {
				e.printStackTrace();

			}
			client.send(jsonObj.toString());
		}
	}

** 3. Save and get Chat History from Sqlite database  **

 *Save Chat history into database*

                                        for (Message ver : listMessages) {
						sqliteDB.execSQL("INSERT INTO " + tableName + " Values ('" + ver.getFromName() + "','"
								+ ver.getMessage() + "','" + ver.isSelf() + "','" + ver.getColor() + "');");
					}

*Get Chat history from Database*

	private void getChatFromDatabase() {
		sqliteDB = null;
		sqliteDB = this.openOrCreateDatabase(dbName, MODE_PRIVATE, null);
		sqliteDB.execSQL("CREATE TABLE IF NOT EXISTS " + tableName
				+ " (userName VARCHAR, message VARCHAR, itself VARCHAR, color VARCHAR);");
		Cursor cursor = sqliteDB.rawQuery("SELECT DISTINCT * FROM " + tableName, null);
		if (cursor != null) {
			if (cursor.moveToFirst()) {
				do {
					String userName = cursor.getString(cursor.getColumnIndex("userName"));
					String message = cursor.getString(cursor.getColumnIndex("message"));
					String isself = cursor.getString(cursor.getColumnIndex("itself"));
					String color = cursor.getString(cursor.getColumnIndex("color"));
					Message msg = new Message();
					msg.setFromName(userName);
					msg.setMessage(message);
					msg.setSelf(Boolean.parseBoolean(isself));
					msg.setColor(color);

					listMessages.add(msg);
				} while (cursor.moveToNext());
			}
		}
	}

**4. Chat history backup from SDCard**

                     public List<Message> getChatFromSdCard(Activity  activity,final File yourFile, final String tableName) {
		final List<Message> list = new ArrayList<Message>();
		activity.runOnUiThread(new Runnable() {
			@Override
			public void run() {
				SQLiteDatabase db = SQLiteDatabase.openOrCreateDatabase(yourFile, null);
				Cursor cursor1 = db.rawQuery("SELECT DISTINCT * FROM " + tableName, null);
				if (cursor1 != null) {
					if (cursor1.moveToFirst()) {
						do {
							String userName = cursor1.getString(cursor1.getColumnIndex("userName"));
							String message = cursor1.getString(cursor1.getColumnIndex("message"));
							String isself = cursor1.getString(cursor1.getColumnIndex("itself"));
							String color = cursor1.getString(cursor1.getColumnIndex("color"));
							Message msg = new Message();
							msg.setFromName(userName);
							msg.setMessage(message);
							msg.setSelf(Boolean.parseBoolean(isself));
							msg.setColor(color);

							list.add(msg);
							System.out.println("Message: " + msg.getFromName() + " " + msg.getMessage());
						} while (cursor1.moveToNext());
					}
				}
			}
		});
		return list;
	}


![Screenshot_2015-03-15-00-13-53.png](https://bitbucket.org/repo/L8qLbo/images/175251175-Screenshot_2015-03-15-00-13-53.png)

![Screenshot_2015-03-15-00-17-59.png](https://bitbucket.org/repo/L8qLbo/images/382379674-Screenshot_2015-03-15-00-17-59.png)

![Screenshot_2015-03-15-00-18-30.png](https://bitbucket.org/repo/L8qLbo/images/4125175617-Screenshot_2015-03-15-00-18-30.png)
