# Access to Third Party Services

Cocos Creator provides a **Service** panel in the **Menu bar -> Panel**, and developers can quickly integrate services through the **Service** panel for the game.

![](index/cocos_services.png)

The **Service** Panel currently supports the integration of third-party services including the following:

  - [Cocos SDKHub](sdkhub.md)

  - [Location Kit (HMS Core)](hms-location.md)

  - [Analytics Kit (HMS Core)](hms-analytics.md)

  - [Agora Voice](https://docs.agora.io/en/Interactive%20Gaming/game_c?platform=Cocos%20Creator)

## Usage

- Open the Cocos Creator, choose **Menu bar -> Panel -> Service** to open the **Service** panel. Click the ![](index/setting.png) button above the **Service** panel. Select **Dashboard** and go to the [Cocos Account Center](https://auth.cocos.com/#/) to register your user account.

  ![](index/console.png)

  Create Personal/Company games as needed after account registration is complete:

  ![](index/game.png)

- After the game is created, return to the Cocos Creator **Service** panel and click the ![](index/setting.png) button. After selecting **Set Cocos AppID**, jump to the **Set Cocos AppID** panel. Then select the game and click the **Association** button.

  ![](index/appid.png)

  You can create a game by clicking the **Cocos AppID is not found. Click here to create** button to jump to the Cocos Account Center.

- When the AppID setup is complete, it automatically jumps to the **Service** panel, where you can see that the game name and AppID are already displayed at the top left of the panel.

  ![](index/service.png)

  If you need to switch games, you can click the ![](index/setting.png) button again to select **Unlink**. Then go to the **Set Cocos AppID** panel again and re-select the game.

  ![](index/switch_appid.png)
