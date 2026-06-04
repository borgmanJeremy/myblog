+++ 
title = "Integrating Sunpower System in Home Assistant"
date = 2026-06-03
author = "Jeremy Borgman"
tags = ['Home Assistant', 'sunpower', 'solar']
+++

This post is still a draft and needs review.

# Introduction
Someone recently asked me how to set up monitoring for sunpower solar panels in home assistant with the legacy PVS6 monitor. Thankfully when SunStrong acquired Sunpower they created an offical API to support something like this: https://github.com/SunStrong-Management/pvs-hass.

However if you are new to home assistant this can still be a daunting task. The steps below comprehensively document step by step how to go from a clean HA install to having the monitoring up and running. 


The first step is to install [HACS](https://www.hacs.xyz/docs/use/download/download/) as a third party [add on](https://www.home-assistant.io/common-tasks/os/#installing-a-third-party-app-repository). 


First navigate to add a custom repository.
![Ha app](/post/solar_homeassistant/1.png)
![Install App](/post/solar_homeassistant/2.png)
![Repositories](/post/solar_homeassistant/3.png)

Then use this URL for the custom repository: https://github.com/hacs/addons 
![dashboard](/post/solar_homeassistant/4.png)


Now that the custom respository is added, you can find HACS and install it.
![dashboard](/post/solar_homeassistant/5.png)
![dashboard](/post/solar_homeassistant/6.png)
![dashboard](/post/solar_homeassistant/7.png)
![dashboard](/post/solar_homeassistant/8.png)

Start HACS and restart home assistant
![dashboard](/post/solar_homeassistant/9.png)
![dashboard](/post/solar_homeassistant/10.png)

Then add the HACS Integration (not 100% sure you must do this step but I recorded exactly what I did)
![dashboard](/post/solar_homeassistant/11.png)
![dashboard](/post/solar_homeassistant/12.png)


Now the PVS plugin can be installed as a custom integration. The upstream documentation for this can be found [here](https://github.com/SunStrong-Management/pvs-hass)

This is the text for the custom add on SunStrong-Management/pvs-hass 
![dashboard](/post/solar_homeassistant/13.png)
![dashboard](/post/solar_homeassistant/14.png)

![dashboard](/post/solar_homeassistant/15.png)
![dashboard](/post/solar_homeassistant/16.png)
![dashboard](/post/solar_homeassistant/17.png)

![dashboard](/post/solar_homeassistant/18.png)
![dashboard](/post/solar_homeassistant/19.png)
![dashboard](/post/solar_homeassistant/20.png)

The integration should be complete now. If you hit devices from the main dashboard you will see all panels and monitors.