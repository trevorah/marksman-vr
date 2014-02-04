marksman-vr
===========
A shooting game written for a virtual reality environment.

There are various functions that help keep the game slightly realistic. 

triggerCheck()
	This function was originally called to test if the trigger button was pressed, and thus call the shoot() function. However, we came across a problem with this method quite quickly. If the function does not take into account if the trigger press event is new or not, and so if the trigger is held, the gun could shoot at every frame. Although a rapid fire gun seems like a fun idea, this was not one of our aims. Even if the user wanted to shoot only once, the gun would shoot around 5-7 times.

	To Counteract this, we used a global boolean value to store whether a virtual bullet was loaded or not. Once the trigger is released, a bullet is loaded, and the gun can only shoot once it is loaded.

shoot()
	One of the problems we had was the opposite of the triggerCheck() problem. When the gun shoots, a bang should be heard. This can be done by calling .play() for a sound object. However, if the gun is fired agian before the sound is over, then calling .play() is pointless (the sound is already playing). Rather than creating multiple sound objects when the gun is fired and playing each one individually, we found a rather straightforward approach. 
We found that stopping the soundobject with .stop() and then starting it again was perfect. We could have simply cut the source wav file down to a smaller size, but we found that it was not as convincing without a the trailing noise. The trailing noise when using the stop/sart method is kept only with the most recent shot, but as a loud bang covers the previous trailing noise anyway, this did not matter.

  
As the bang sound's main impact is the opening staccato 
```
function triggerCheck(){
// The trigger needs to be the correct joystick button
	var trigger = keyPressed("g");
    if (!trigger){
    	barrelLoaded = true;
	}
    if (trigger && barrelLoaded){
    	shoot();
        barrelLoaded = false;
	}
}

function shoot()
{
	output_text = "Shot registered! ";
	GunSound.setVolume(1.0);
    
    shotCounter++;

    //we set the range of the shot (length of shooting vector
    //usually the vector is of unit length - so we need to extend it
    var range = 250;

    //we are using the camera as the gun - need to get handtracker pos and direction
	var posVector = CameraGetPosition();
    var aimVector = CameraGetDirection();

    //show the bullet trace
    bulletTraceTimerCounter = 0;
    //TracerObj.setPosition(posVector+aimVector*300);
       
    //iterate through list of targets and test if the vector collides
    for(var i = 0; i < len(targetList); i++){
		var target = targetList[i];
        if(target.isColliding(posVector,posVector+aimVector*range)){
            // Lower volume of gun so we can hear the hit...
            GunSound.SetVolume(0.7);
            CanHit.play();
            hitCounter++;
            output_text = output_text + "\nHit object number " + str(i+1) + "\n";
            outputln(output_text);
            randVisableCanID = (rand() % 8);
            while(randVisableCanID==visableCanID)
            	randVisableCanID = (rand() % 8);
            visableCanID = randVisableCanID;
		}
	}
	//Makes sure the BANG is heard...
	if(GunSound.isPlaying())
		GunSound.stop();
	GunSound.play();
    var accuracy = (hitCounter*100)/shotCounter;
    output_text = output_text + "Total hits = " + str(hitCounter) + ", accuracy = " + str(accuracy) + "%";
    outputln(output_text);
}
```
