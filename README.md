//Variables
var clientspawn : GameObject;
var playerPref : GameObject;
var dirrectcon = false;
var nickname : String = "";
private var nickname1 : String = "";
var ip : String = "127.0.0.1";
var port : String = "25001";
var started = false;
var enableGUI = false;
var starting = false;
public var gamename : String = "";
private var disconnecting = false;
var myChar : GameObject;

var gameName : String = "TDGTutorial";
private var refreshing : boolean = false;
private var hosts : HostData[];

function OnGUI() { //Gui drawing function
	if(nickname == "" || nickname == null){ //Asks you to enter your nickname before showing any other options
		if(enableGUI == true){
			if (GUI.Button(Rect(Screen.width/2-50,Screen.height/2+25,100,20),"Continue")) {
		    	nickname = nickname1;
		    	Debug.Log(nickname);
			}
			GUI.TextArea(Rect(Screen.width/2-50,Screen.height/2-25,100,20),"Enter nickname");
			nickname1 = GUI.TextField(Rect(Screen.width/2-50,Screen.height/2,100,20), nickname1, 25);
		}
	}
	else if(nickname != null){ //Another menu which appears right after entering nickname
		if(started == false && starting == false && enableGUI == true && dirrectcon == false){
			if(!Network.isClient && !Network.isServer) {
				if (GUI.Button(Rect(Screen.width/2-50,Screen.height/2,100,20),"Start Server")) {
	    	    	starting = true;
				}
				if (GUI.Button(Rect(Screen.width/2-50,Screen.height/2 - 30,100,20),"Direct connect")) {
	    	    	dirrectcon = true;
				}
			 
				if (GUI.Button(Rect(Screen.width/2-50,Screen.height/2 + 30,100,20),"Refresh Hosts")) {
			        Debug.Log("Refresh");
			        refresh();
				}
			        if(hosts) {
	 
	       			for(var i:int = 0; i<hosts.length; i++) { //This is a for loop which shows every detected host
	       
	        		if(GUI.Button(Rect(Screen.width/2-50,Screen.height/2 + 60 + 30 * i,100,20),hosts[i].gameName)) {
	                	Network.Connect(hosts[i]);
	       				}
	       			}
	        	}
			}
		}
		else if(started == false && starting == true && enableGUI == true && dirrectcon == false){ //Another menu which asks to set the name of the game when pressing start server
			if(GUI.Button(Rect(Screen.width/2-50,Screen.height/2 + 60,100,20),"Back")) {
	    	   	started = false;
	    	   	starting = false;
			}
			gamename = GUI.TextField(Rect(Screen.width/2-50,Screen.height/2 + 30,100,20), gamename, 25);
			if(GUI.Button(Rect(Screen.width/2-50,Screen.height/2,100,20),"Start")) {
	    	   	startServer();
	    	}
		}
		else if(started == true && disconnecting == false && enableGUI == true){ //This appears when the server is completely running
			if (GUI.Button(Rect(Screen.width/2-50,Screen.height/2,100,20),"Disconnect")) {
	    		disconnecting = true;
			}
			
		}
		else if(started == true && disconnecting == true && enableGUI == true){ //Confirmation screen about disconnecting from server
			if (GUI.Button(Rect(Screen.width/2-50,Screen.height/2 + 30,100,20),"Yes")) {
	    		disconnect();
			}
			if (GUI.Button(Rect(Screen.width/2-50,Screen.height/2,100,20),"No")) {
	    		disconnecting = false;
			}	
		}
		else if(dirrectcon == true && enableGUI == true){ //Used to connect to the server outside LAN - Online
			if(GUI.Button(Rect(Screen.width/2-50,Screen.height/2 + 30,100,20),"Back")) {
	    		   	started = false;
	    		   	starting = false;
	    		   	dirrectcon = false;
				}
				ip = GUI.TextField(Rect(Screen.width/2-100,Screen.height/2 - 30,100,20), ip, 25);
				port = GUI.TextField(Rect(Screen.width/2,Screen.height/2 - 30,100,20), port, 25);
				if(GUI.Button(Rect(Screen.width/2-50,Screen.height/2,100,20),"Start")) {
	    		   	joinIP();
	   		}
	   	}
	}
}

 	function Update () { //Function called every frame
	if(Input.GetKeyDown(KeyCode.Escape) && enableGUI == true)
		enableGUI = false;
	else if(Input.GetKeyDown(KeyCode.Escape) && enableGUI == false)
		enableGUI = true;
		
	if(enableGUI == true){
	gameObject.GetComponent(MP_Menu).enabled = true;
	}
	if(refreshing == true) {
    	if(MasterServer.PollHostList().Length > 0) {           
        	refreshing = false;
        	Debug.Log(MasterServer.PollHostList().Length);
        	hosts = MasterServer.PollHostList();          
		}
	}
}

function startServer(){ //starts the server
	started = true;
	Debug.Log(gamename);
	Network.InitializeServer(8,System.Int32.Parse(port));
    MasterServer.RegisterHost(gameName, gamename, " this is a tutorial");
}
function joinIP(){
	Network.Connect(ip, System.Int32.Parse(port));
}
function refresh(){ //refreshes host list
	MasterServer.RequestHostList(gameName);
    refreshing = true;
}

function OnServerInitialized(){ //This is used once server is started
	enableGUI = false;
	spawnPlayer();
}
function OnConnectedToServer(){ //This is used when client successfully connects to server
	enableGUI = false;
	started = true;
    starting = true;
	spawnPlayer();
}
function disconnect(){ //Disconnect from server
	Network.Destroy(myChar);
	Network.Disconnect();
	MasterServer.UnregisterHost();
	started = false;
	starting = false;
	Application.LoadLevel(Application.loadedLevel);
}
function refreshHostList () { //Refresh host list in LAN
	MasterServer.RequestHostList(gameName);
	refreshing = true;     
} 
function spawnPlayer() { //spawns player in spawn gameobject position
	myChar = Network.Instantiate(playerPref, clientspawn.transform.position, Quaternion.identity, 0);
}
function OnMasterServerEvent(mse:MasterServerEvent) {
	if(mse == MasterServerEvent.RegistrationSucceeded) {
    	Debug.Log("Registered Server");
	}  
}
