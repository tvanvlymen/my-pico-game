# my-pico-game
<html><head>
<title>PICO-8 Cartridge</title>
<meta name="viewport" content="width=device-width, user-scalable=no">
<script type="text/javascript">

	// Default shell for PICO-8 0.2.2 (includes @weeble's gamepad mod 1.0)
	// This file is available under a CC0 license https://creativecommons.org/share-your-work/public-domain/cc0/
	// (note: "this file" does not include any cartridge or cartridge artwork injected into a derivative html file when using the PICO-8 html exporter)

	// options

	// fullscreen, sound, close button at top when playing on touchscreen
	var p8_allow_mobile_menu = true;

	// p8_autoplay true to boot the cartridge automatically after page load when possible
	// if the browser can not create an audio context outside of a user gesture (e.g. on iOS), p8_autoplay has no effect
	var p8_autoplay = false;

	// When pico8_state is defined, PICO-8 will set .is_paused, .sound_volume and .frame_number each frame 
	// (used for determining button icons)
	var pico8_state = [];

	// When pico8_buttons is defined, PICO-8 reads each int as a bitfield holding that player's button states
	// 0x1 left, 0x2 right, 0x4 up, 0x8 right, 0x10 O, 0x20 X, 0x40 menu
	// (used by p8_update_gamepads)
	var pico8_buttons = [0, 0, 0, 0, 0, 0, 0, 0]; // max 8 players

	// When pico8_mouse is defined, PICO-8 reads the 3 integers as X, Y and a bitfield for buttons: 0x1 LMB, 0x2 RMB
	var pico8_mouse = [];

	// used to display number of detected joysticks
	var pico8_gamepads = {};
	pico8_gamepads.count = 0;

	// When pico8_gpio is defined, reading and writing to gpio pins will read and write to these values
	var pico8_gpio = new Array(128);

	// When pico8_audio_context context is defined, the html shell (this file) is responsible for creating and managing it.
	// This makes satisfying browser requirements easier -- e.g. initialising audio from a short script in response to a user action.
	// Otherwise PICO-8 will try to create and use its own context.

	var pico8_audio_context;


	// menu button and controller graphics
	p8_gfx_dat={
			"p8b_pause1": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAAOUlEQVRIx2NgGPbg/8cX/0F46FtAM4vobgHVLRowC6hm0YBbQLFFoxaM4FQ0dHPy0C1Nh26NNugBAAnizNiMfvbGAAAAAElFTkSuQmCC",
"p8b_controls":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAAQ0lEQVRIx2NgGAXEgP8fX/ynBaap4XBLhqcF1IyfYWQBrZLz0LEAlzqqxQFVLcAmT3MLqJqTaW7B4CqLaF4fjIIBBwBL/B2vqtPVIwAAAABJRU5ErkJggg==",
"p8b_full":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAAN0lEQVRIx2NgGPLg/8cX/2mJ6WcBrUJm4CwgOSgGrQVEB8WoBaMWDGMLhm5OHnql6dCt0YY8AAA9oZm+9Z9xQAAAAABJRU5ErkJggg==",
"p8b_pause0":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAAKUlEQVRIx2NgGHbg/8cX/7FhctWNWjBqwagFoxaMWjBqwagF5Fkw5AAAPaGZvsIUtXUAAAAASUVORK5CYII=",
"p8b_sound0":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAANklEQVRIx2NgGDHg/8cX/5Hx0LEA3cChYwEugwavBcRG4qgFoxYMZwuGfk4efqXp8KnRBj0AAMz7cLDnG4FeAAAAAElFTkSuQmCC",
"p8b_sound1":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAAPUlEQVRIx2NgGDHg/8cX/5Hx0LEA3cChYwEugwhZQLQDqG4BsZFIKMhGLRi1YChbMPRz8vArTYdPjTboAQCSVgpXUWQAMAAAAABJRU5ErkJggg==",
"p8b_close":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAAU0lEQVRIx2NkoDFgpJsF/z+++I8iwS9BkuW49A+cBcRaREgf/Swg1SJi1dHfAkIG4EyOOIJy4Cwg1iJCiWDUAvItGLqpaOjm5KFfmg79Gm3ItioAl+mAGVYIZUUAAAAASUVORK5CYII=",

"controls_left_panel":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAASwAAAEsCAYAAAB5fY51AAAEI0lEQVR42u3dMU7DQBCG0Tjam9DTcP8jpEmfswS5iHBhAsLxev/hvQY6pGXyZRTQ+nQCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHqbHEEtl+vt7hS+fLy/mXHBQqxEi/6aI/AiFW9SnB2BWDkDBAtAsADBAhAsAMECBAtAsAAECxAsAMECECxAsAAEC0CwONJ8tYvrXRAsImK19j0IFsPGSrQQLCJiNV+et7xAT7QQLIaN1dr3ooVgMWysRAvBIipWooVgERUr0UKwiIqVaCFYRMVKtBAsomIlWggWUbESLQSLqFiJFoJFVKxEC8EiKlaihWARFSvRQrDYJSSVfhaCBSBYAIIFCBbAHpoj4Bl/scOGBWDD4lX8iwE2LADBAgQLQLAABAsQLADBAhAsQLAABAtAsADBAhAsAMECBAtAsAAECxAsAMECECxAsAAECxAsAMECECxAsMh1ud7uTsHZVDcZyFo8Yt5sVJ6NyUAaSNEyIymaXwZepIKd4mwoQbAFC0CwAMECECwAwQIEC0CwAAQLECwAwQIQLECwAAQLQLAAwQI4UHME2/10QZq7usyBObBhRQwpmBUb1nADuPbuaUD/p2ezMH+1admwhosVfBcxb2SCJVaIlmAhVoiWYIkVoiVagiVWiJZgiZVYIVqCJVaIlmgJllghWoIlViBagiVWiJZoCZZYIVqCJVYgWoIlViBaggUIlnc0sPELlmghVmIlWKKFWAmWaIFYCZZoIVYIlmghVoIlWiBWgiVaiJVgIVqIlWCJFoiVYIkWYiVYiBZiJViihViJ1XbNEWyL1mMQRYvfvIGJlQ1rmE0LzIoNyyBiDrBhAYIFIFiAYAEIFoBgAYIFIFgAggUIFoBgAQgWIFgAggUgWIBgDc+Nn1D/tdH8YupwgZy5qG4ykKIlVmZDsDjshSlazqQqH7p793Q2CBaAYAGCBSBYAIIFCBaAYAEIFiBYAIIFIFiAYAEIFoBgAYIFIFgAggUIFoBgAQgWIFgAggUgWIBgAQgWwENzBKxZPub9CJ7WjA0LsGFRV+9N5+jNDhsWgGABggUgWACCxW56fgjuA3cEiz9Z/nWwR0iWP8P/YCFYDBstsUKwiIiWWCFYRERLrBAsIqIlVggWEdESKwSLiGiJFYJFRLTECsEiIlpihWARES2xQrCIiJZYIVhEREusECwioiVWCBYx0RIrBIuoaIkVr+YhFHTZtMCGBQgWgGABCBYgWACCBSBYgGABCBaAYAGCBSBYAIIFCBbj2uOR8s6AEbhexgsWYri3SKhKczcXAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMA2n+e0UMDzh3yTAAAAAElFTkSuQmCC",


"controls_right_panel":"data:image/png;base64,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",

	};


	// added 0.2.1: work-around for iOS/Safari running from an iFrame (e.g. from itch.io page):
	// touch events only register after adding dummy listeners on document.

	document.addEventListener('touchstart', {});
	document.addEventListener('touchmove', {});
	document.addEventListener('touchend', {});


	// --------------------------------------------------------------------------------------------------------------------------------
	// pico-8 0.2.2: allow dropping files
	var p8_dropped_cart = null;
	var p8_dropped_cart_name = "";
	function p8_drop_file(e)
	{
		// console.log("@@ dropping file...");
	
		e.stopPropagation();
		e.preventDefault();

		if (e.dataTransfer && e.dataTransfer.files && e.dataTransfer.files[0])
		{
			// read from file
			reader = new FileReader();
			reader.onload = function (event)
			{
				p8_dropped_cart_name = 'untitled.p8';
				if (typeof e.dataTransfer.files[0].name !== 'undefined') p8_dropped_cart_name = e.dataTransfer.files[0].name;
				if (typeof e.dataTransfer.files[0].fileName !== 'undefined') p8_dropped_cart_name = e.dataTransfer.files[0].fileName;
				p8_dropped_cart = reader.result;
				// data:image/png;base64
				e.stopPropagation();
				e.preventDefault();
				codo_command = 9; // read directly from p8_dropped_cart with libb64 decoder
			};
			reader.readAsDataURL(e.dataTransfer.files[0]);
			
		}
		else
		{
			// read from url (or data url)
			txt = e.dataTransfer.getData('Text');
			if (txt){
				p8_dropped_cart_name = "untitled.p8.png";
				p8_dropped_cart = txt;
				codo_command = 9;
			}
		}
	}
	function nop(evt) {
		evt.stopPropagation();
		evt.preventDefault();
	}
	function dragover(evt) {
		evt.stopPropagation();
		evt.preventDefault();
		Module.pico8DragOver();
	}
	function dragstop(evt) {
		evt.stopPropagation();
		evt.preventDefault();
		Module.pico8DragStop();
	}

	// download (pico-8 0.2.4d web exports can save a .wav file)
	function download_browser_file(filename, contents)
	{
	  var element = document.createElement('a');
	  if (filename.substr(filename.length - 7) == ".p8.png")
	  	element.setAttribute('href', 'data:image/png;base64,' + encodeURIComponent(contents));
	  else if (filename.substr(filename.length - 4) == ".wav")
	  	element.setAttribute('href', 'data:audio/x-wav;base64,' + encodeURIComponent(contents));
	  else
	  	element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(contents));
	  element.setAttribute('download', filename);
	  element.style.display = 'none';
	  document.body.appendChild(element);
	  element.click();
	  document.body.removeChild(element);
	}
	// --------------------------------------------------------------------------------------------------------------------------------


	var p8_buttons_hash = -1;
	function p8_update_button_icons()
	{
		// buttons only appear when running
		if (!p8_is_running)
		{
			requestAnimationFrame(p8_update_button_icons);
			return;
		}
		var is_fullscreen=(document.fullscreenElement || document.mozFullScreenElement || document.webkitIsFullScreen || document.msFullscreenElement);
		
		// hash based on: pico8_state.sound_volume  pico8_state.is_paused bottom_margin left is_fullscreen p8_touch_detected
		var hash = 0;
		hash = pico8_state.sound_volume;
		if (pico8_state.is_paused) hash += 0x100;
		if (p8_touch_detected)     hash += 0x200;
		if (is_fullscreen)         hash += 0x400;

		if (p8_buttons_hash == hash)
		{
			requestAnimationFrame(p8_update_button_icons);
			return;
		}
		
		p8_buttons_hash = hash;
		// console.log("@@ updating button icons");

		els = document.getElementsByClassName('p8_menu_button');
		for (i = 0; i < els.length; i++)
		{
			el = els[i];
			index = el.id;			
			if (index == 'p8b_sound') index += (pico8_state.sound_volume == 0 ? "0" : "1"); // 1 if undefined
			if (index == 'p8b_pause') index += (pico8_state.is_paused > 0 ? "1" : "0");     // 0 if undefined

			new_str = '<img width=24 height=24 style="pointer-events:none" src="'+p8_gfx_dat[index]+'">';
			if (el.innerHTML != new_str)
				el.innerHTML = new_str;




			// hide all buttons for touch mode (can pause with menu buttons)
			
			var is_visible = p8_is_running;

			if ((!p8_touch_detected || !p8_allow_mobile_menu) && el.parentElement.id == "p8_menu_buttons_touch")
				is_visible = false;

			if (p8_touch_detected && el.parentElement.id == "p8_menu_buttons")
				is_visible = false;

			if (is_fullscreen) 
				is_visible = false;

			if (is_visible)
				el.style.display="";
			else
				el.style.display="none";
		}
		requestAnimationFrame(p8_update_button_icons);
	}



	function abs(x)
	{
		return x < 0 ? -x : x;
	}
	
	// step 0 down 1 drag 2 up (not used)
	function pico8_buttons_event(e, step)
	{
		if (!p8_is_running) return;
	
		pico8_buttons[0] = 0;

		if (step == 2 && typeof(pico8_mouse) !== 'undefined')
		{
			pico8_mouse[2] = 0;
		}
		
		var num = 0;
		if (e.touches) num = e.touches.length;

		if (num == 0 && typeof(pico8_mouse) !== 'undefined')
		{
			//  no active touches: release mouse button from anywhere on page. (maybe redundant? but just in case)
			pico8_mouse[2] = 0;
		}

		
		for (var i = 0; i < num; i++)
		{
			var touch = e.touches[i];
			var x = touch.clientX;
			var y = touch.clientY;
			var w = window.innerWidth;
			var h = window.innerHeight;

			var r = Math.min(w,h) / 12;
			if (r > 40) r = 40;

			// mouse (0.1.12d)

			let canvas = document.getElementById("canvas");
			if (p8_touch_detected)
			if (typeof(pico8_mouse) !== 'undefined')
			if (canvas)
			{
				var rect = canvas.getBoundingClientRect();
				//console.log(rect.top, rect.right, rect.bottom, rect.left, x, y);

				if (x >= rect.left && x < rect.right && y >= rect.top && y < rect.bottom)
				{
					pico8_mouse = [
						Math.floor((x - rect.left) * 128 / (rect.right - rect.left)),
						Math.floor((y - rect.top) * 128 / (rect.bottom - rect.top)),
						step < 2 ? 1 : 0
					];
					// return; // commented -- blocks overlapping buttons
				}else
				{
					pico8_mouse[2] = 0;
				}
			}


			// buttons
			
			b = 0;

			if (y < h - r*8)
			{
				// no controller buttons up here; includes canvas and menu buttons at top in touch mode
			}
			else
			{
				e.preventDefault();

				if ((y < h - r*6) && y > (h - r*8))
				{
					// menu button: half as high as X O button
					// stretch across right-hand half above X O buttons
					if (x > w - r*3) 
						b |= 0x40;
				}
				else if (x < w/2 && x < r*6)
				{
					// stick

					mask = 0xf; // dpad
					var cx = 0 + r*3;
					var cy = h - r*3;

					deadzone = r/3;
					var dx = x - cx;
					var dy = y - cy;

					if (abs(dx) > abs(dy) * 0.6) // horizontal 
					{
						if (dx < -deadzone) b |= 0x1;
						if (dx > deadzone) b |= 0x2;
					}
					if (abs(dy) > abs(dx) * 0.6) // vertical
					{
						if (dy < -deadzone) b |= 0x4;
						if (dy > deadzone) b |= 0x8;
					}
				}
				else if (x > w - r*6)
				{
					// button; diagonal split from bottom right corner
				
					mask = 0x30;
					
					// one or both of [X], [O]
					if ( (h-y) > (w-x) * 0.8) b |= 0x10;
					if ( (w-x) > (h-y) * 0.8) b |= 0x20;
				}
			}

			pico8_buttons[0] |= b;
		
		}
	}

	// p8_update_layout_hash is used to decide when to update layout (expensive especially when part of a heavy page)
	var p8_update_layout_hash = -1;
	var last_windowed_container_height = 512;
	var p8_layout_frames = 0;

	function p8_update_layout()
	{
		var canvas = document.getElementById("canvas");
		var p8_playarea = document.getElementById("p8_playarea");
		var p8_container = document.getElementById("p8_container");
		var p8_frame = document.getElementById("p8_frame");
		var csize = 512;
		var margin_top = 0;
		var margin_left = 0;

		// page didn't load yet? first call should be after p8_frame is created so that layout doesn't jump around.
		if (!canvas || !p8_playarea || !p8_container || !p8_frame)
		{
			p8_update_layout_hash = -1;
			requestAnimationFrame(p8_update_layout);
			return;
		}

		p8_layout_frames ++;

		// assumes frame doesn't have padding
		
		var is_fullscreen=(document.fullscreenElement || document.mozFullScreenElement || document.webkitIsFullScreen || document.msFullscreenElement);
		var frame_width = p8_frame.offsetWidth;
		var frame_height = p8_frame.offsetHeight;

		if (is_fullscreen)
		{
			// same as window
			frame_width = window.innerWidth;
			frame_height = window.innerHeight;
		}
		else{
			// never larger than window  // (happens when address bar is down in portraight mode on phone)
			frame_width  = Math.min(frame_width, window.innerWidth);
			frame_height = Math.min(frame_height, window.innerHeight);
		}

		// as big as will fit in a frame..
		csize =  Math.min(frame_width,frame_height);

		// .. but never more than 2/3 of longest side for touch (e.g. leave space for controls on iPad)
		if (p8_touch_detected && p8_is_running)
		{
			var longest_side = Math.max(window.innerWidth,window.innerHeight);
			csize = Math.min(csize, longest_side * 2/3);
		}

		// pixel perfect: quantize to closest multiple of 128
		// only when large display (desktop)
		if (frame_width >= 512 && frame_height >= 512)
		{
			csize = (csize+1) & ~0x7f;
		}

		// csize should never be higher than parent frame
		// (otherwise stretched large when fullscreen and then return)
		if (!is_fullscreen && p8_frame) 
			csize = Math.min(csize, last_windowed_container_height); // p8_frame_0 parent
		

		if (is_fullscreen)
		{
			// always center horizontally
			margin_left = (frame_width - csize)/2;

			if (p8_touch_detected)
			{
				if (window.innerWidth < window.innerHeight)
				{
					// portrait: keep at y=40 (avoid rounded top corners / camera nub thing etc.)
					margin_top = Math.min(40, frame_height - csize);
				}
				else
				{
					// landscape: put a little above vertical center
					margin_top = (frame_height - csize)/4;
				}
			}
			else{
				// non-touch: center vertically
				margin_top = (frame_height - csize)/2;
			}
		}

		// skip if relevant state has not changed

		var update_hash = csize + margin_top * 1000.3 + margin_left * 0.001 + frame_width * 333.33 + frame_height * 772.15134;
		if (is_fullscreen) update_hash += 0.1237;

		// unexpected things can happen in the first few seconds, so just keep re-calculating layout. wasm version breaks layout otherwise.
		// also: bonus refresh at 5, 8 seconds just in case ._.
		if (p8_layout_frames < 180 || p8_layout_frames == 60*5 || p8_layout_frames == 60*8 )
			update_hash = p8_layout_frames;

		if (!is_fullscreen) // fullscreen: update every frame for safety. should be cheap!
		if (!p8_touch_detected) // mobile: update every frame because nothing can be trusted
		if (p8_update_layout_hash == update_hash)
		{
			//console.log("p8_update_layout(): skipping");
			requestAnimationFrame(p8_update_layout);
			return;
		}
		p8_update_layout_hash = update_hash;

		// record this for returning to original size after fullscreen pushes out container height (argh)
		if (!is_fullscreen && p8_frame)
			last_windowed_container_height = p8_frame.parentNode.parentNode.offsetHeight;
		

		// mobile in portrait mode: put screen at top (w / a little extra space for fullscreen button if needed)
		// (don't cart too about buttons overlapping screen)
		if (p8_touch_detected && p8_is_running && document.body.clientWidth < document.body.clientHeight)
			p8_playarea.style.marginTop = p8_allow_mobile_menu ? 32 : 8;
		else if (p8_touch_detected && p8_is_running) // landscape: slightly above vertical center (only relevant for iPad / highres devices)
			p8_playarea.style.marginTop = (document.body.clientHeight - csize) / 4;
		else
			p8_playarea.style.marginTop = "";

		canvas.style.width = csize;
		canvas.style.height = csize;

		// to do: this should just happen from css layout
		canvas.style.marginLeft = margin_left;
		canvas.style.marginTop = margin_top;

		p8_container.style.width = csize;
		p8_container.style.height = csize;

		// set menu buttons position to bottom right
		el = document.getElementById("p8_menu_buttons");
		el.style.marginTop = csize - el.offsetHeight;

		if (p8_touch_detected && p8_is_running)
		{
			// turn off pointer events to prevent double-tap zoom etc (works on Android)
			// don't want this for desktop because breaks mouse input & click-to-focus when using codo_textarea
			canvas.style.pointerEvents = "none";

			p8_container.style.marginTop = "0px";

			// buttons
			
			// same as touch event handling
			var w = window.innerWidth;
			var h = window.innerHeight;
			var r = Math.min(w,h) / 12;

			if (r > 40) r = 40;

			el = document.getElementById("controls_right_panel");
			el.style.left = w-r*6;
			el.style.top = h-r*7;
			el.style.width = r*6;
			el.style.height = r*7;
			if (el.getAttribute("src") != p8_gfx_dat["controls_right_panel"]) // optimisation: avoid reload? (browser should handle though)
				el.setAttribute("src", p8_gfx_dat["controls_right_panel"]);

			el = document.getElementById("controls_left_panel");
			el.style.left = 0;
			el.style.top = h-r*6;
			el.style.width = r*6;
			el.style.height = r*6;
			if (el.getAttribute("src") != p8_gfx_dat["controls_left_panel"]) // optimisation: avoid reload? (browser should handle though)
				el.setAttribute("src", p8_gfx_dat["controls_left_panel"]);
			
			// scroll to cart (commented; was a failed attempt to prevent scroll-on-drag on some browsers)
			// p8_frame.scrollIntoView(true);

			document.getElementById("touch_controls_gfx").style.display="table";
			document.getElementById("touch_controls_background").style.display="table";

		}
		else{
			document.getElementById("touch_controls_gfx").style.display="none";
			document.getElementById("touch_controls_background").style.display="none";
		}

		if (!p8_is_running)
		{
			p8_playarea.style.display="none";
			p8_container.style.display="flex";
			p8_container.style.marginTop="auto";

			el = document.getElementById("p8_start_button");
			if (el) el.style.display="flex";
		}
		requestAnimationFrame(p8_update_layout);
	}


	var p8_touch_detected = false;
	addEventListener("touchstart", function(event)
	{
		p8_touch_detected = true;

		// hide codo_textarea -- clipboard support on mobile is not feasible
		el = document.getElementById("codo_textarea");
		if (el && el.style.display != "none"){
			el.style.display="none";
		}

	},  {passive: true});

	function p8_create_audio_context()
	{
		if (pico8_audio_context) 
		{
			try {
				pico8_audio_context.resume();
			}
			catch(err) {
				console.log("** pico8_audio_context.resume() failed");
			}	
			return;
		}

		var webAudioAPI = window.AudioContext || window.webkitAudioContext || window.mozAudioContext || window.oAudioContext || window.msAudioContext;			
		if (webAudioAPI)
		{
			pico8_audio_context = new webAudioAPI;

			// wake up iOS
			if (pico8_audio_context)
			{
				try {
					var dummy_source_sfx = pico8_audio_context.createBufferSource();
					dummy_source_sfx.buffer = pico8_audio_context.createBuffer(1, 1, 22050); // dummy
					dummy_source_sfx.connect(pico8_audio_context.destination);
					dummy_source_sfx.start(1, 0.25); // gives InvalidStateError -- why? hasn't been played before 
					//dummy_source_sfx.noteOn(0); // deleteme
				}
				catch(err) {
					console.log("** dummy_source_sfx.start(1, 0.25) failed");
				}
			}
		}
	}

	function p8_close_cart()
	{
		// just reload page! used for touch buttons -- hard to roll back state 
		window.location.hash = ""; // triggers reload
	}

	var p8_is_running = false;
	var p8_script = null;
	var Module = null;
	function p8_run_cart()
	{
		if (p8_is_running) return;
		p8_is_running = true;

		// touch: hide everything except p8_frame_0
		if (p8_touch_detected)
		{
			el = document.getElementById("body_0");
			el2 = document.getElementById("p8_frame_0");
			if (el && el2)
			{
				el.style.display="none";
				el.parentNode.appendChild(el2);
			}
		}

		// create audio context and wake it up (for iOS -- needs happen inside touch event)		
		p8_create_audio_context();

		// show touch elements
		els = document.getElementsByClassName('p8_controller_area');
		for (i = 0; i < els.length; i++)
			els[i].style.display="";


		// install touch events. These also serve to block scrolling / pinching / zooming on phones when p8_is_running
			// moved event.preventDefault(); calls into pico8_buttons_event() (want to let top buttons pass through)
		addEventListener("touchstart", function(event){ pico8_buttons_event(event, 0); }, {passive: false});
		addEventListener("touchmove",  function(event){ pico8_buttons_event(event, 1); }, {passive: false});
		addEventListener("touchend",   function(event){ pico8_buttons_event(event, 2); }, {passive: false});


		// load and run script
		e = document.createElement("script");
		p8_script = e;
		e.onload = function(){
			
			// show canvas / menu buttons only after loading
			el = document.getElementById("p8_playarea");
			if (el) el.style.display="table";

			if (typeof(p8_update_layout_hash) !== 'undefined')
				p8_update_layout_hash = -77;
			if (typeof(p8_buttons_hash) !== 'undefined')
				p8_buttons_hash = -33;


		}
		e.type = "application/javascript";
		e.src = "roguelike.js";
		e.id = "e_script";
		
		document.body.appendChild(e); // load and run

		// hide start button and show canvas / menu buttons. hide start button
		el = document.getElementById("p8_start_button");
		if (el) el.style.display="none";

		// add #playing for touchscreen devices (allows back button to close)
		// X button can also be used to trigger this 
		if (p8_touch_detected)
		{
			window.location.hash = "#playing";
			window.onhashchange = function()
			{
				if (window.location.hash.search("playing") < 0)
					window.location.reload();
			}
		}

		// install drag&drop listeners
		{
			let canvas = document.getElementById("canvas");
			if (canvas)
			{
				canvas.addEventListener('dragenter', dragover, false);
				canvas.addEventListener('dragover', dragover, false);
				canvas.addEventListener('dragleave', dragstop, false);
				canvas.addEventListener('drop', nop, false);
				canvas.addEventListener('drop', p8_drop_file, false);
			}
		}
	}

	
		// Gamepad code
	
	var P8_BUTTON_O = {action:'button', code: 0x10};
	var P8_BUTTON_X = {action:'button', code: 0x20};
	var P8_DPAD_LEFT = {action:'button', code: 0x1};
	var P8_DPAD_RIGHT = {action:'button', code: 0x2};
	var P8_DPAD_UP = {action:'button', code: 0x4};
	var P8_DPAD_DOWN = {action:'button', code: 0x8};
	var P8_MENU = {action:'menu'};
	var P8_NO_ACTION = {action:'none'};

	var P8_BUTTON_MAPPING = [
		// ref: https://w3c.github.io/gamepad/#remapping
		P8_BUTTON_O,          // Bottom face button
		P8_BUTTON_X,          // Right face button
		P8_BUTTON_X,          // Left face button
		P8_BUTTON_O,          // Top face button
		P8_NO_ACTION,         // Near left shoulder button (L1)
		P8_NO_ACTION,         // Near right shoulder button (R1)
		P8_NO_ACTION,         // Far left shoulder button (L2)
		P8_NO_ACTION,         // Far right shoulder button (R2)
		P8_MENU,              // Left auxiliary button (select)
		P8_MENU,              // Right auxiliary button (start)
		P8_NO_ACTION,         // Left stick button
		P8_NO_ACTION,         // Right stick button
		P8_DPAD_UP,           // Dpad up
		P8_DPAD_DOWN,         // Dpad down
		P8_DPAD_LEFT,         // Dpad left
		P8_DPAD_RIGHT,        // Dpad right
	];

	// Track which player is controller by each gamepad. Gamepad index i controls the
	// player with index pico8_gamepads_mapping[i]. Gamepads with null player are
	// currently unassigned - they get assigned to a player when a button is pressed.
	var pico8_gamepads_mapping = [];

	function p8_unassign_gamepad(gamepad_index) {
		if (pico8_gamepads_mapping[gamepad_index] == null) {
			return;
		}
		pico8_buttons[pico8_gamepads_mapping[gamepad_index]] = 0;
		pico8_gamepads_mapping[gamepad_index] = null;
	}


	function p8_first_player_without_gamepad(max_players) {
		var allocated_players = pico8_gamepads_mapping.filter(function(x) { return x != null; });
		var sorted_players = Array.from(allocated_players).sort();
		for (var desired = 0; desired < sorted_players.length && desired < max_players; ++desired) {
			if (desired != sorted_players[desired]) {
				return desired;
			}
		}
		if (sorted_players.length < max_players) {
			return sorted_players.length;
		}
		return null;
	}

	function p8_assign_gamepad_to_player(gamepad_index, player_index) {
		p8_unassign_gamepad(gamepad_index);
		pico8_gamepads_mapping[gamepad_index] = player_index;
	}



	function p8_convert_standard_gamepad_to_button_state(gamepad, axis_threshold, button_threshold) {
		// Given a gamepad object, return:
		// {
		//     button_state: the binary encoded Pico 8 button state
		//     menu_button: true if any menu-mapped button was pressed
		//     any_button: true if any button was pressed, including d-pad
		//         buttons and unmapped buttons
		// }
		if (!gamepad || !gamepad.axes || !gamepad.buttons) {
			return {
				button_state: 0,
				menu_button: false,
				any_button: false
			};
		}
		function button_state_from_axis(axis, low_state, high_state, default_state) {
			if (axis && axis < -axis_threshold) return low_state;
			if (axis && axis > axis_threshold) return high_state;
			return default_state;
		}
		var axes_actions = [
			button_state_from_axis(gamepad.axes[0], P8_DPAD_LEFT, P8_DPAD_RIGHT, P8_NO_ACTION),
			button_state_from_axis(gamepad.axes[1], P8_DPAD_UP, P8_DPAD_DOWN, P8_NO_ACTION),
		];

		var button_actions = gamepad.buttons.map(function (button, index) {
			var pressed = button.value > button_threshold || button.pressed;
			if (!pressed) return P8_NO_ACTION;
			return P8_BUTTON_MAPPING[index] || P8_NO_ACTION;
		});

		var all_actions = axes_actions.concat(button_actions);
		
		var menu_button = button_actions.some(function (action) { return action.action == 'menu'; });
		var button_state = (all_actions
			.filter(function (a) { return a.action == 'button'; })
			.map(function (a) { return a.code; })
			.reduce(function (result, code) { return result | code; }, 0)
		);
		var any_button = gamepad.buttons.some(function (button) {
			return button.value > button_threshold || button.pressed;
		});

		any_button |= button_state; //jww: include axes 0,1 as might be first intended action

		return {
			button_state,
			menu_button,
			any_button
		};
	}

	// jww: pico-8 0.2.1 version for unmapped gamepads, following p8_convert_standard_gamepad_to_button_state
	// axes 0,1 & buttons 0,1,2,3 are reasonably safe. don't try to read dpad.
	// menu buttons are unpredictable, but use 6..8 anyway (better to have a weird menu button than none)

	function p8_convert_unmapped_gamepad_to_button_state(gamepad, axis_threshold, button_threshold) {

		if (!gamepad || !gamepad.axes || !gamepad.buttons) {
			return {
				button_state: 0,
				menu_button: false,
				any_button: false
			};
		}

		var button_state = 0;

		if (gamepad.axes[0] && gamepad.axes[0] < -axis_threshold) button_state |= 0x1;
		if (gamepad.axes[0] && gamepad.axes[0] > axis_threshold)  button_state |= 0x2;
		if (gamepad.axes[1] && gamepad.axes[1] < -axis_threshold) button_state |= 0x4;
		if (gamepad.axes[1] && gamepad.axes[1] > axis_threshold)  button_state |= 0x8;

		// buttons: first 4 taken to be O/X, 6..8 taken to be menu button

		for (j = 0; j < gamepad.buttons.length; j++)
		if (gamepad.buttons[j].value > 0 || gamepad.buttons[j].pressed)
		{
			if (j < 4)
				button_state |= (0x10 << (((j+1)/2)&1)); // 0 1 1 0 -- A,X -> O,X on xbox360
			else if (j >= 6 && j <= 8)
				button_state |= 0x40;
		}

		var menu_button = button_state & 0x40;

		var any_button = gamepad.buttons.some(function (button) {
			return button.value > button_threshold || button.pressed;
		});

		any_button |= button_state; //jww: include axes 0,1 as might be first intended action
		
		return {
			button_state,
			menu_button,
			any_button
		};
	}

	
	// gamepad  https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API/Using_the_Gamepad_API
	// (sets bits in pico8_buttons[])
	function p8_update_gamepads() {
		var axis_threshold = 0.3;
		var button_threshold = 0.5; // Should be unnecessary, we should be able to trust .pressed
		var max_players = 8;
		var gps = navigator.getGamepads() || navigator.webkitGetGamepads();

		if (!gps) return;

		// In Chrome, gps is iterable but it's not an array.
		gps = Array.from(gps);

		pico8_gamepads.count = gps.length;
		while (gps.length > pico8_gamepads_mapping.length) {
		    pico8_gamepads_mapping.push(null);
		}

		var menu_button = false;
		var gamepad_states = gps.map(function (gp) {
			return (gp && gp.mapping == "standard") ?
				p8_convert_standard_gamepad_to_button_state(gp, axis_threshold, button_threshold) :
				p8_convert_unmapped_gamepad_to_button_state(gp, axis_threshold, button_threshold);
		});

		// Unassign disconnected gamepads.
		// gps.forEach(function (gp, i) { if (gp && !gp.connected) { p8_unassign_gamepad(i); }});
		gps.forEach(function (gp, i) { if (!gp || !gp.connected) { p8_unassign_gamepad(i); }}); // https://www.lexaloffle.com/bbs/?pid=87132#p


		// Assign unassigned gamepads when any button is pressed.
		gamepad_states.forEach(function (state, i) {
			if (state.any_button && pico8_gamepads_mapping[i] == null) {
				var first_free_player = p8_first_player_without_gamepad(max_players);
				p8_assign_gamepad_to_player(i, first_free_player);
			}
		});

		// Update pico8_buttons array.
		gamepad_states.forEach(function (gamepad_state, i) {
			if (pico8_gamepads_mapping[i] != null) {
				pico8_buttons[pico8_gamepads_mapping[i]] = gamepad_state.button_state;
			}
		});

		// Update menu button.
		// Pico 8 only recognises the menu button on the first player, so we
		// press it when any gamepad has pressed a button mapped to menu.
		if (gamepad_states.some(function (state) { return state.menu_button; })) {
			pico8_buttons[0] |= 0x40;
		}

		requestAnimationFrame(p8_update_gamepads);
	}
	requestAnimationFrame(p8_update_gamepads);

	// End of gamepad code


	// key blocker. prevent browser operations while playing cart so that PICO-8 can use those keys e.g. cursors to scroll, ctrl-r to reload
	document.addEventListener('keydown',
	function (event) {
		event = event || window.event;
		if (!p8_is_running) return;

		if (pico8_state.has_focus == 1)
			if ([32, 37, 38, 39, 40, 77, 82, 80, 9].indexOf(event.keyCode) > -1)       // block only cursors, M R P, tab
				if (event.preventDefault) event.preventDefault();

	},{passive: false});

	// when using codo_textarea to determine focus, need to explicitly hand focus back when clicking a p8_menu_button
	function p8_give_focus()
	{
		el = (typeof codo_textarea === 'undefined') ? document.getElementById("codo_textarea") : codo_textarea;
		if (el)
		{
			el.focus();
			el.select();
		}
	}

	function p8_request_fullscreen() {

		var is_fullscreen=(document.fullscreenElement || document.mozFullScreenElement || document.webkitIsFullScreen || document.msFullscreenElement);

		if (is_fullscreen)
		{
			 if (document.exitFullscreen) {
		        document.exitFullscreen();
		    } else if (document.webkitExitFullscreen) {
		        document.webkitExitFullscreen();
		    } else if (document.mozCancelFullScreen) {
		        document.mozCancelFullScreen();
		    } else if (document.msExitFullscreen) {
		        document.msExitFullscreen();
		    }
			return;
		}
		
		var el = document.getElementById("p8_playarea");

		if ( el.requestFullscreen ) {
			el.requestFullscreen();
		} else if ( el.mozRequestFullScreen ) {
			el.mozRequestFullScreen();
		} else if ( el.webkitRequestFullScreen ) {
			el.webkitRequestFullScreen( Element.ALLOW_KEYBOARD_INPUT );
		}
	}

</script>

<STYLE TYPE="text/css">
<!--
.p8_menu_button{
	opacity:0.3;
	padding:4px;
	display:table;
	width:24px;
	height:24px;
	float:right;
}

@media screen and (min-width:512px) {
	.p8_menu_button{
		width:24px; margin-left:12px; margin-bottom:8px;
	}
}
.p8_menu_button:hover{
	opacity:1.0;
	cursor:pointer;
}

canvas{
    image-rendering: optimizeSpeed;
    image-rendering: -moz-crisp-edges;
    image-rendering: -webkit-optimize-contrast;
    image-rendering: optimize-contrast;
    image-rendering: pixelated;
    -ms-interpolation-mode: nearest-neighbor;
	border: 0px;
	cursor: none;
}


.p8_start_button{
	cursor:pointer;
	background:url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAIAAAACACAIAAABMXPacAAAEYklEQVR4Ae2dP47bPBDFHcO1m43PIBgudCbBVwgW6RbpUnxXEHiFNLlBqhQBXBgLn2G7PUEKbgbzDUlpKFuUSL4fhAV3LInimz+v3QAAAKiXl69fIAJyUC+fKAffvv+nfKbrzvoNjOmh8mIdEJUqqI8EZELXne0lhA7FwfSyJU25skLWUBwJ0LC9pwn0cTB9cLuDZbjwMYKi2I3eYUzfdWdj+pDE9BPknqUDVvIqeABIOIJQuQAAAAAAAAAAAAAgMV13thetxULckyO77L7YmL6kIttm2hYYD6AyD9gfjqP3YATNPvG9aSjMFVaKlX5/ONICmqQYQfSX0iAy4d4D/jeXvYsJcVHybgI0i9F4+kTulprmA3EbsX+FIlb097fX97dXnm96iTG9eES/b+EmHKovNy4qlEO6UweIl7iP6PcFYAbzfJQTtKeGj2n77wPfX5SZu4eJnQPeOIlO/z7w/UslYHYPcM/pPepofLPZXK437syX6+2x768FXrmauEhke2pEK8y6b/YjaMKx+U+0HihV7/2x+xbuAfcXIJlw7OhQ7lt+Au6fGKIbxA13TpKiEgAy7gAaNWQhfKGPV90B7mGi5oMQkftqVHz4C4XWtXgAV2dUqZAcynjo/V4LgQd86EUStKeGl3ZsfCD3rvrwgKHhMC0+7WvTd8A2cYGPxv/8/sWFIH1j48pB13Vn+wZ0gH9W8HERG4/dq3YPSO83tXhAVFlZI6VJQr66Pxy9V+j+xF273gRMOJIQ0Varldu78N4f9amFJyB0qlGl+INWa4EIRtmvMG14wIdeJEF7atx5wstfiO69v6K5P6EDhktyuNgpGPv+KjrAe5hQAYo4maoQiM993geh+2P3hQl7pLFqkgmH2sJ7f7KvrQVe+6IPgGoE6cvfFrJ9nHw1Np7ym7PxAOXxxJ00UmLjSMBHSXrdVaNUSA5lXGm/BSZgdLwM6EK/tqeGl3ZsXKn75I7J1QOsTMPPeh+MjWMEaQ8j8iG6RLirPh5b+IUnoOvO+k6nO62a3EWi4vV2AMhpBFX+zds02+wPR3utRO61fc+81cTPyddKe3x43Ps9hXuApuJS5qPGDljPmb3fU7gHrKTwa8GtpvWbcOEeABbuAHzzwh4QGkFzj6al9l1XNfFzataPstmofYv1AHtCqjhRd5pKnJaP0L7D31NgB/ATigMLLeYYPu6+oXhRHeCOoFD5DyTg/kE02nY8XnICsjDhYj0ALNMB9qK1WOjj9O/oXgOv1e+bvgN2yXYyplfGbcT+VSpCLzGmF4/o912EbeK20MR5SXKZQvZrRQ8Ncf2+YNzMOTYfME9QWQeIQZR7B6TzgHtywLlcb6P3AFCBCWMELUB7augqo8hySoAQvZgc5MTL89PL85NdoAMW8IAfPz8L9XP3gF1en3u53jab5t8CHbDoLMJAXkx32G86D7AXrcVC3AMPSIExfUlFlp8HZF3vAAAAAAAAAAAAAAAAAAAAAAAAAEjGX7okYKAyTjgpAAAAEHRFWHRMb2RlUE5HADIwMTEwMjIx41m2wQAAAABJRU5ErkJggg==");
	-repeat center;
	-webkit-background-size:cover; -moz-background-size:cover; -o-background-size:cover; background-size:cover;
}

.button_gfx{
	stroke-width:2;
	stroke: #ffffff;
	stroke-opacity:0.4;
	fill-opacity:0.2;
	fill:black;
}

.button_gfx_icon{
	stroke-width:3;
	stroke: #909090;
	stroke-opacity:0.7;
	fill:none;
}

-->
</STYLE>

</head>

<body style="padding:0px; margin:0px; background-color:#222; color:#ccc">
<div id="body_0"> <!-- hide this when playing in mobile (p8_touch_detected) so that elements don't affect layout -->


<!-- Add any content above the cart here -->


<div id="p8_frame_0" style="max-width:800px; max-height:800px; margin:auto;"> <!-- double function: limit size, and display only this div for touch devices -->
<div id="p8_frame" style="display:flex; width:100%; max-width:95vw; height:100vw; max-height:95vh; margin:auto;">

	<div id="p8_menu_buttons_touch" style="position:absolute; width:100%; z-index:10; left:0px;">
		<div class="p8_menu_button" id="p8b_full"  style="float:left;margin-left:10px" onClick="p8_give_focus(); p8_request_fullscreen();"></div>
		<div class="p8_menu_button" id="p8b_sound" style="float:left;margin-left:10px" onClick="p8_give_focus(); p8_create_audio_context(); Module.pico8ToggleSound();"></div>
		<div class="p8_menu_button" id="p8b_close" style="float:right; margin-right:10px" onClick="p8_close_cart();"></div>
	</div>

	<div id="p8_container"
		style="margin:auto; display:table;"
		onclick="p8_create_audio_context(); p8_run_cart();">

		<div id="p8_start_button" class="p8_start_button" style="width:100%; height:100%; display:flex;">
			<img width=80 height=80 style="margin:auto;"
		src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFAAAABQCAYAAACOEfKtAAABpklEQVR42u3au23DQBCEYUXOXIGKcujQXUgFuA0XIKgW90Q9oEAg+Ljd27vd2RsCf058gEDqhofPj+OB6SMCAQlIQAIyAhKQgARkBAQDnM6XSRsB7/2e/tSA0//12fCAKsQX3ntDA4oRFwBRIc0AixE38BAhTQGLEAsBUSDNAXcRhYDRIZsAPlp99VECRoXsDpgN0g0wC6Q7IDpkGEBUyG6A0+vKBtkdMBukG2AWSHdAdMgwgKiQ4QDRIMMCokCGB4wOCQPYFVKw2cABNocUjl6wgE0gFashPKAZpHJ2TQNYBVmxW6cDFENWDv9pAUshCVgJScBKSAISkD9hPkT4GkNAMdzepyj8Kye852EBLe51CZHHWQK4JcThD1SlcHPEYY/0a+A0n6SkGZV6w6WZNb3g4Id1b7hwgGhwYQBR4dwB0eHcALPAdQfMBhcOEA0uDCAqnDsgOpwbYBa4poA/31+rZYFrBriFpwGMCtcEcA9PAhgdzhywBK8EEQXOFFCCtwaIBmcGKMWbI6LCmQBq8R6hw5kAMgISkIAEJCAjIAEJSEBGQI9ukV7lRn9nD+gAAAAASUVORK5CYII="/>
		</div>

		<div id="p8_playarea" style="display:none; margin:auto;
				-webkit-user-select:none; -moz-user-select: none; user-select: none; -webkit-touch-callout:none;
		">

			<div  id="touch_controls_background"
				  style=" pointer-events:none; display:none; background-color:#000;
						 position:fixed; top:0px; left:0px; border:0; width:100vw; height:100vh">
				&nbsp
			</div>

			<div style="display:flex; position:relative">
				<!-- pointer-events turned off for mobile in p8_update_layout because need for desktop mouse -->
				<canvas class="emscripten" id="canvas" oncontextmenu="event.preventDefault();" >
				</canvas>
				<div class=p8_menu_buttons id="p8_menu_buttons" style="margin-left:10px;">
					<div class="p8_menu_button" style="position:absolute; bottom:125px" id="p8b_controls" onClick="p8_give_focus(); Module.pico8ToggleControlMenu();"></div>					
					<div class="p8_menu_button" style="position:absolute; bottom:90px" id="p8b_pause" onClick="p8_give_focus(); Module.pico8TogglePaused(); p8_update_layout_hash = -22;"></div>
					<div class="p8_menu_button" style="position:absolute; bottom:55px" id="p8b_sound" onClick="p8_give_focus(); p8_create_audio_context(); Module.pico8ToggleSound();"></div>
					<div class="p8_menu_button" style="position:absolute; bottom:20px" id="p8b_full" onClick="p8_give_focus(); p8_request_fullscreen();"></div>
				</div>
			</div>


			<!-- display after first layout update -->
			<div  id="touch_controls_gfx"
				  style=" pointer-events:none; display:table; 
						 position:fixed; top:0px; left:0px; border:0; width:100vw; height:100vh">

					<img src="" id="controls_right_panel" style="position:absolute; opacity:0.5;">
					<img src="" id="controls_left_panel" style="position:absolute;  opacity:0.5;">
						
			
			</div> <!-- touch_controls_gfx -->

			<!-- used for clipboard access & keyboard input; displayed and used by PICO-8 only once needed. can be safely removed if clipboard / key presses not needed. -->
			<!-- (needs to be inside p8_playarea so that it still works under Chrome when fullscreened) -->
			<!-- 0.2.5: added "display:none"; pico8.js shows on demand to avoid mac osx accent character selector // https://www.lexaloffle.com/bbs/?tid=47743 -->

			<textarea id="codo_textarea" class="emscripten" style="display:none; position:absolute; left:-9999px; height:0px; overflow:hidden"></textarea>

		</div> <!--p8_playarea -->

	</div> <!-- p8_container -->

</div> <!-- p8_frame -->
</div> <!-- p8_frame_0 size limit -->

<script type="text/javascript">

	p8_update_layout();
	p8_update_button_icons();

	var canvas = document.getElementById("canvas");
	Module = {};
	Module.canvas = canvas;

	// from @ultrabrite's shell: test if an AudioContext can be created outside of an event callback.
	// If it can't be created, then require pressing the start button to run the cartridge

	if (p8_autoplay)
	{
		var temp_context = new AudioContext();
		temp_context.onstatechange = function ()
		{
			if (temp_context.state=='running')
			{
				p8_run_cart();
				temp_context.close();
			}
		};
	}

	// pointer lock request needs to be inside a canvas interaction event
	// pico8_state.request_pointer_lock is true when 0x5f2d bit 0 and bit 2 are set -- poke(0x5f2d,0x5) 
	// note on mouse acceleration for future: // https://github.com/w3c/pointerlock/pull/49
	canvas.addEventListener("click", function()
	{
		if (!p8_touch_detected)
			if (pico8_state.request_pointer_lock)
				canvas.requestPointerLock();
	});
	
</script>



<!-- Add content below the cart here -->




</div> <!-- body_0 -->
</body></html>
