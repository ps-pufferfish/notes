# Look Better
 - smooth movement
	 - separate game behaviour from game display
	 - underlying snake model - take from now
	 - separate snake ui
	 - keyboard input defines 'target head location' which informs ui

 - Snake wireframe calculated after each gameplay loop.
	 - Used for ui in between keyframes
 - Keyboard used to know which direction to smoothly move towards
	 - Potentially have some lerping between old keyboard and new keyboard position
 - Gameplay loop defines 'keyframes', ui updates happen faster
 - Move Threading out of GameplayControler
# New Features
 - Score
 - leaderboard

# Learning Outcomes
 - Design patters
 - testing practices



## Snake plan
 - 