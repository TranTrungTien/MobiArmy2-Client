# MobiArmy2 - System Architecture & Codebase Map (Client & Server)

> Tài liệu này được biên soạn làm Index điều hướng & phân tích mã nguồn cho AI hoặc Developer để phát hiện bug, đối chiếu packet giao tiếp, và tối ưu logic mà không cần nạp toàn bộ mã nguồn cùng lúc.

## 1. Tổng quan Dự án (Project Overview)
- **Tên dự án**: MobiArmy 2 (Bản Port LibGDX + Server Emulation)
- **Mục tiêu**: Tái hiện game tọa độ huyền thoại MobiArmy 2 từ nền tảng J2ME sang đa nền tảng (Desktop/Android/iOS qua LibGDX) và Server Java Socket.
- **Cơ chế chính**: Tính toán góc, lực, gió, trọng lực, va chạm địa hình phá hủy (destructible map), quản lý trang bị nhân vật, boss chiến đấu theo lượt.
- **Công nghệ sử dụng**:
  - **Client**: Java 8, LibGDX 1.12.1, LWJGL3 (Desktop), RoboVM (iOS), Box2D.
  - **Server**: Java ServerSocket, H2 Database (tương thích MySQL), Multi-threading Session.

## 2. Luồng giao tiếp cốt lõi (Client - Server Flow)
```
Client (Session_ME) <--- Binary Packet (CMD byte) ---> Server (Session/MessageHandler)
  - CMD 1: Đăng nhập (Login)
  - CMD 20: Bắt đầu trận đấu / Nạp dữ liệu phòng (Start Game / Training)
  - CMD 21, 24, 25: Di chuyển, Lượt bắn, Đồng bộ góc/lực
  - CMD 83: Rời trận đấu / Kết thúc luyện tập (Leave Game / End Training)
  - CMD 105: Mua bán / Thao tác trang bị, VIP Equip (Cmd_Client2Server)
```

## 3. Cấu trúc thư mục (Directory Structure)
```
MobiArmy2-Client/core/src/
├── CLib/                 # Tầng Wrapper giả lập J2ME -> LibGDX (Graphics, Image, Sound, Socket)
├── com/teamobi/mobiarmy2 # Khởi chạy LibGDX (MainGame, TemCanvas, Android/Desktop bridge)
├── coreLG/               # Core game loop, Canvas điều khiển chung (CCanvas, TerrainMidlet)
├── effect/               # Hiệu ứng đồ họa (Khói, Nổ, Tuyết, Lốc xoáy, Camera)
├── Equipment/            # Dữ liệu & render trang bị nhân vật (Equip, PlayerEquip)
├── item/                 # Quản lý đạn bay (Bullet, BM) và item hỗ trợ
├── map/                  # Bản đồ, background cuộn, tính năng phá hủy map (Background, CMap, MM)
├── model/                # Entity, cấu hình Font, Dialog, Dữ liệu nhân vật (PlayerInfo)
├── network/              # Socket & xử lý gói tin (MessageHandler, GameService, Session_ME)
├── player/               # Điều khiển nhân vật, góc ngắm, animation (CPlayer, Boss, PM)
├── screen/               # Màn hình giao diện (GameScr, EquipScreen, PrepareScr, MenuScr)
└── shop/                 # Giao diện mua sắm đồ, biệt đội, item

MobiArmy2-Server/src/mobiarmy/
├── server/               # Quản lý Server, Session, DBManager, User, Shop, MessageHandler
├── war/                  # Động cơ phòng đấu, phòng chờ, lượt bắn, mô phỏng vật lý
│   └── Boss/             # AI các loại Boss và đường đạn đặc thù (Bullet logic)
└── io/                   # Đọc/ghi Message nhị phân qua Socket
```

## 4. Chi tiết Client Classes (169 files)

| File / Class | Package | Mục đích | Các Method chính |
|---|---|---|---|
| **`EasingEnum`**<br>`CLib/EasingEnum.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | - |
| **`EasingFunction`**<br>`CLib/EasingFunction.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `Linear(start, end, value): float`<br>`Spring(start, end, value): float`<br>`EaseInQuad(start, end, value): float`<br>`EaseOutQuad(start, end, value): float`<br>`EaseInOutQuad(start, end, value): float`<br>*(+59 methods khác)* |
| **`Graphics`**<br>`CLib/Graphics.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | - |
| **`Image`**<br>`CLib/Image.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `createImage(url): Image`<br>`run(): void`<br>`createImage(url, action2): void`<br>`run(): void`<br>`getWidth(): int`<br>*(+27 methods khác)* |
| **`LibSysTem`**<br>`CLib/LibSysTem.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `getResourceAsStream(path): InputStream`<br>`openWeb(links): void` |
| **`Library_Images`**<br>`CLib/Library_Images.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | - |
| **`Main`**<br>`CLib/Main.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `exit(): void` |
| **`ManagerTile`**<br>`CLib/ManagerTile.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `loadTileBig(id): void`<br>`loadTileBigLogin(id): void`<br>`freeBigTile(id): void`<br>`freeBigTile1(id): void` |
| **`MapKey`**<br>`CLib/MapKey.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `load(): void`<br>`map(kyeCode): int` |
| **`RMS`**<br>`CLib/RMS.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `saveRMSInt(file, x): void`<br>`saveRMS(filename, data): void`<br>`loadRMS(filename): byte[]`<br>`loadRMSInt(file): int`<br>`saveRMSString(filename, s): void`<br>*(+4 methods khác)* |
| **`ScrollResult`**<br>`CLib/ScrollResult.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | - |
| **`SoundSystem`**<br>`CLib/SoundSystem.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `play(volume): void`<br>`isPlaying(): boolean`<br>`pause(): void`<br>`stop(): void`<br>`setLooping(isLoop2): void`<br>*(+1 methods khác)* |
| **`Sound_Key`**<br>`CLib/Sound_Key.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | - |
| **`Sprite`**<br>`CLib/Sprite.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | - |
| **`SysTemFont`**<br>`CLib/SysTemFont.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `rgba8888ToColor(rgba8888): Color`<br>`getWidth(st): int`<br>`convert_RGB_to_ARGB(rgb): int`<br>`replace(_text, _searchStr, _replacementStr): String`<br>`splitFont(src, lineWidth): String[]`<br>*(+4 methods khác)* |
| **`TField`**<br>`CLib/TField.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `doChangeToTextBox(): void`<br>`input(text): void`<br>`canceled(): void`<br>`getHeight(): int`<br>`positionLogin(): void`<br>*(+27 methods khác)* |
| **`TemGraphics`**<br>`CLib/TemGraphics.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | - |
| **`mFont`**<br>`CLib/mFont.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `loadmFont(): void`<br>`setColoFont(id): int`<br>`reloadImage(): void`<br>`freeImage(): void`<br>`getHeight(): int`<br>*(+8 methods khác)* |
| **`mGraphics`**<br>`CLib/mGraphics.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `translate(tx, ty): void`<br>`begin(): void`<br>`end(): void`<br>`getTranslateX(): int`<br>`getTranslateY(): int`<br>*(+51 methods khác)* |
| **`mHashtable`**<br>`CLib/mHashtable.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `clear(): void`<br>`contains(obj): boolean`<br>`containsKey(obj): boolean`<br>`get(k): Object`<br>`size(): int`<br>*(+3 methods khác)* |
| **`mImage`**<br>`CLib/mImage.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `getLink(str): String`<br>`replaceImg(str): String`<br>`createImage(url): mImage`<br>`createImage(url, iAction2): void`<br>`createImageAll(url): mImage`<br>*(+15 methods khác)* |
| **`mSocket`**<br>`CLib/mSocket.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `close(): void`<br>`setKeepAlive(isAlive): void`<br>`getOutputStream(): DataOutputStream`<br>`getInputStream(): DataInputStream`<br>`getIP(): String`<br>*(+1 methods khác)* |
| **`mSound`**<br>`CLib/mSound.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `init(): void`<br>`getSoundPoolSource(index, fileName): int`<br>`playSound(id, volume, index): void`<br>`SetLoopSound(id, volume, loop): void`<br>`UnSetLoopAll(): void`<br>*(+10 methods khác)* |
| **`mSystem`**<br>`CLib/mSystem.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `currentTimeMillis(): long`<br>`deltaTime(): float`<br>`my_Gc(): void`<br>`new_M_Int(value1, value2): int[][]`<br>`new_M_String(value1, value2): String[][]`<br>*(+33 methods khác)* |
| **`mVector`**<br>`CLib/mVector.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `addElement(o): void`<br>`contains(o): boolean`<br>`size(): int`<br>`elementAt(index): Object`<br>`setElementAt(obj, index): void`<br>*(+8 methods khác)* |
| **`myEditText`**<br>`CLib/myEditText.java` | `CLib` | Tầng abstraction chuyển đổi J2ME graphics/sound/socket sang LibGDX. | `end(): void` |
| **`GameMidlet`**<br>`com/teamobi/mobiarmy2/GameMidlet.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | `initGame(): void`<br>`doUpdateServer(): void`<br>`perform(object): void`<br>`saveIP(): void`<br>`loadIP(): void`<br>*(+18 methods khác)* |
| **`IActionListener`**<br>`com/teamobi/mobiarmy2/IActionListener.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | - |
| **`InAppPurchaseSystem`**<br>`com/teamobi/mobiarmy2/InAppPurchaseSystem.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | - |
| **`IosVersion`**<br>`com/teamobi/mobiarmy2/IosVersion.java` | `default` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | - |
| **`MainActivity`**<br>`com/teamobi/mobiarmy2/MainActivity.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | `makePurchase(string): void` |
| **`MainClass`**<br>`com/teamobi/mobiarmy2/MainClass.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | `create(): void`<br>`perform(object): void`<br>`render(): void`<br>`dispose(): void`<br>`initaliseInputProcessors(): void`<br>*(+9 methods khác)* |
| **`MainGame`**<br>`com/teamobi/mobiarmy2/MainGame.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | `create(): void`<br>`initaliseInputProcessors(): void`<br>`render(): void`<br>`getWidth(): int`<br>`getHeight(): int`<br>*(+26 methods khác)* |
| **`MotherCanvas`**<br>`com/teamobi/mobiarmy2/MotherCanvas.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | `displayMe(m): void`<br>`getHeightL(): int`<br>`getWidthL(): int`<br>`setFullScreenMode(i): void`<br>`start(): void`<br>*(+8 methods khác)* |
| **`MyGdxGame`**<br>`com/teamobi/mobiarmy2/MyGdxGame.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | `create(): void`<br>`render(): void`<br>`dispose(): void`<br>`resize(width, height): void`<br>`pause(): void`<br>*(+1 methods khác)* |
| **`NativeController`**<br>`com/teamobi/mobiarmy2/NativeController.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | - |
| **`TemCanvas`**<br>`com/teamobi/mobiarmy2/TemCanvas.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | `start(): void`<br>`paint(gx): void`<br>`update(): void`<br>`keyPressed(keyCode): void`<br>`keyReleased(keyCode): void` |
| **`TemMidlet`**<br>`com/teamobi/mobiarmy2/TemMidlet.java` | `com.teamobi.mobiarmy2` | Entry point LibGDX và cầu nối nền tảng Desktop/Android/iOS. | `destroyApp(arg0): void`<br>`makePurchase(productId): void`<br>`pauseApp(): void`<br>`startApp(): void`<br>`destroy(): void`<br>*(+9 methods khác)* |
| **`CCanvas`**<br>`coreLG/CCanvas.java` | `coreLG` | Module chức năng CCanvas. | `screenInit(setV): void`<br>`loadScreen(): void`<br>`getGameAction(keyCode): int`<br>`mainLoop(): void`<br>`update(): void`<br>*(+68 methods khác)* |
| **`CONFIG`**<br>`coreLG/CONFIG.java` | `coreLG` | Module chức năng CONFIG. | - |
| **`SynchronizedData`**<br>`coreLG/SynchronizedData.java` | `coreLG` | Module chức năng SynchronizedData. | `invoke(): void` |
| **`TerrainMidlet`**<br>`coreLG/TerrainMidlet.java` | `coreLG` | Module chức năng TerrainMidlet. | `saveIP(): void`<br>`split(_text, _searchStr): String[]`<br>`connectHTTP(url): String`<br>`destroyApp(arg0): void`<br>`showMyCanvas(): void`<br>*(+7 methods khác)* |
| **`Debug`**<br>`Debug/Debug.java` | `Debug` | Module chức năng Debug. | `gI(): Debug`<br>`setup(): void`<br>`paint(_mGraphic): void`<br>`create_RBGImage(): void`<br>`update(): void`<br>*(+5 methods khác)* |
| **`Camera`**<br>`effect/Camera.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (Camera) trong màn chơi. | `init(_camera): void`<br>`setxy(x, y): void`<br>`setFreeMode(): void`<br>`setPlayerMode(target): void`<br>`setBulletMode(target): void`<br>*(+15 methods khác)* |
| **`Cloud`**<br>`effect/Cloud.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (Cloud) trong màn chơi. | `updateCloud(): void`<br>`balloonUpdate(): void`<br>`paintCloud(g): void`<br>`paintBalloonWithCloud(g): void`<br>`paintSimpleClound(CloundType, x, y, g): void` |
| **`Effect`**<br>`effect/Effect.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (Effect) trong màn chơi. | `Flash(g, x, y, width, height, alpha): void`<br>`DrawBlurImage(g, img, x, y, blurLevel): void`<br>`FillTransparentRect(g, x, y, width, height): void`<br>`FillTransparentRectRGB(g, x, y, width, height, color): void`<br>`fillTransparentRect(g, x, y, width, height, color, transPercent, useDrawRGB): void`<br>*(+2 methods khác)* |
| **`Explosion`**<br>`effect/Explosion.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (Explosion) trong màn chơi. | `perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>*(+10 methods khác)* |
| **`GiftEffect`**<br>`effect/GiftEffect.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (GiftEffect) trong màn chơi. | `paint(g): void`<br>`update(): void` |
| **`Smoke`**<br>`effect/Smoke.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (Smoke) trong màn chơi. | `perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>*(+22 methods khác)* |
| **`SmokeManager`**<br>`effect/SmokeManager.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (SmokeManager) trong màn chơi. | `addSmoke(x, y, Type): void`<br>`addLazer(xF, yF, xT, yT, type): void`<br>`addBat(xF, yF, xT, yT): void`<br>`addRock(x, y, vx, vy, Type): void`<br>`update(): void`<br>*(+3 methods khác)* |
| **`Snow`**<br>`effect/Snow.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (Snow) trong màn chơi. | `startSnow(typeS): void`<br>`update(): void`<br>`paintBigSnow(g): void`<br>`paintOnlySmall(g): void`<br>`paintSmallSnow(g): void` |
| **`Tornado`**<br>`effect/Tornado.java` | `effect` | Xử lý hiệu ứng đồ họa/vật lý (Tornado) trong màn chơi. | `paint(g): void`<br>`update(): void` |
| **`Equip`**<br>`Equipment/Equip.java` | `Equipment` | Quản lý model, dữ liệu và render trang bị (Equip). | `getBaseAttribute(): int[]`<br>`setInvAtribute(): void`<br>`removeAbility(): void`<br>`addAbilityFromEquip(e): void`<br>`getInvAtribute(ability): void`<br>*(+7 methods khác)* |
| **`EquipGlass`**<br>`Equipment/EquipGlass.java` | `Equipment` | Quản lý model, dữ liệu và render trang bị (EquipGlass). | `addType(types): void`<br>`getType(id): TypeEquip` |
| **`PlayerEquip`**<br>`Equipment/PlayerEquip.java` | `Equipment` | Quản lý model, dữ liệu và render trang bị (PlayerEquip). | `addOneEquip(eq): void`<br>`addGlassEquip(glass): void`<br>`getEquipGlass(id): EquipGlass`<br>`getEquip(glass, type, id): Equip`<br>`createEquip(glass, type, id): Equip`<br>*(+7 methods khác)* |
| **`TypeEquip`**<br>`Equipment/TypeEquip.java` | `Equipment` | Quản lý model, dữ liệu và render trang bị (TypeEquip). | `addEquip(equips): void`<br>`getEquip(id): Equip` |
| **`InAppPurchaseSystem`**<br>`InApp/InAppPurchaseSystem.java` | `InApp` | Module chức năng InAppPurchaseSystem. | - |
| **`MainActivity`**<br>`InApp/MainActivity.java` | `InApp` | Module chức năng MainActivity. | `makePurchase(string): void` |
| **`PurchaseView`**<br>`InApp/PurchaseView.java` | `InApp` | Module chức năng PurchaseView. | - |
| **`ChatTextField`**<br>`InterfaceComponents/ChatTextField.java` | `InterfaceComponents` | Module chức năng ChatTextField. | `gI(): ChatTextField`<br>`setChat(type): void`<br>`commandTab(index, subIndex): void`<br>`init(): void`<br>`keyPressed(keyCode): void`<br>*(+4 methods khác)* |
| **`GamePad`**<br>`InterfaceComponents/GamePad.java` | `InterfaceComponents` | Module chức năng GamePad. | `update(i): void`<br>`paint(g): void` |
| **`BM`**<br>`item/BM.java` | `item` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `setBullType(critical, whoShot, type, xPaint, yPaint, nShoot, force2, xHit, yHit, idBullet): void`<br>`onInitSpecialBullet(): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>*(+19 methods khác)* |
| **`Bullet`**<br>`item/Bullet.java` | `item` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>*(+52 methods khác)* |
| **`Item`**<br>`item/Item.java` | `item` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `DrawItem(g, itemID, x, y): void`<br>`drawThisItem(g, x, y): void`<br>`DrawSetItem(g, setItem, curItem, x, y, isTouch, num): void` |
| **`MyItemIcon`**<br>`item/MyItemIcon.java` | `item` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `setIAction(iAction): void`<br>`checkCmtoY(Select): void`<br>`moveCamera(): void`<br>`update(): void`<br>`mainLoop(): void`<br>*(+12 methods khác)* |
| **`RandomItem`**<br>`item/RandomItem.java` | `item` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `newPos(x, y): void`<br>`update(): void`<br>`paint(g): void` |
| **`Graphics`**<br>`javax/microedition/lcdui/Graphics.java` | `javax.microedition.lcdui` | Tầng socket I/O, đóng gói và đọc Message nhị phân. | - |
| **`Image`**<br>`javax/microedition/lcdui/Image.java` | `javax.microedition.lcdui` | Tầng socket I/O, đóng gói và đọc Message nhị phân. | `createImage(url): Image`<br>`getColor(rgb): Color`<br>`getWidth(): int`<br>`getHeight(): int`<br>`createImageTextureRegion(): Image`<br>*(+15 methods khác)* |
| **`MIDlet`**<br>`javax/microedition/midlet/MIDlet.java` | `javax.microedition.midlet` | Tầng socket I/O, đóng gói và đọc Message nhị phân. | `notifyDestroyed(): void`<br>`platformRequest(url): void` |
| **`MIDletStateChangeException`**<br>`javax/microedition/midlet/MIDletStateChangeException.java` | `javax.microedition.midlet` | Tầng socket I/O, đóng gói và đọc Message nhị phân. | - |
| **`Context`**<br>`lib/Context.java` | `lib` | Module chức năng Context. | - |
| **`MyStream`**<br>`lib/MyStream.java` | `lib` | Module chức năng MyStream. | `readFile(path): InputStream` |
| **`TCanvas`**<br>`lib/TCanvas.java` | `lib` | Module chức năng TCanvas. | - |
| **`mGraphics`**<br>`lib/mGraphics.java` | `lib` | Module chức năng mGraphics. | `drawImage(arg0, x, y, arg3): void`<br>`drawImage(arg0, x, y, arg3): void`<br>`drawLine(x1, y1, x2, y2): void`<br>`fillRect(x, y, w, h, color, alpha): void`<br>`drawRect(x, y, w, h): void`<br>*(+25 methods khác)* |
| **`mSocket`**<br>`lib/mSocket.java` | `lib` | Module chức năng mSocket. | - |
| **`Background`**<br>`map/Background.java` | `map` | Quản lý render địa hình, background, phá hủy map (Background). | `perform(object): void`<br>`loadImage(Type): void`<br>`removeImage(): void`<br>`initImage(): void`<br>`update(): void`<br>*(+9 methods khác)* |
| **`CMap`**<br>`map/CMap.java` | `map` | Quản lý render địa hình, background, phá hủy map (CMap). | `onInitCmap(): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>*(+15 methods khác)* |
| **`HoleInfo`**<br>`map/HoleInfo.java` | `map` | Quản lý render địa hình, background, phá hủy map (HoleInfo). | - |
| **`MM`**<br>`map/MM.java` | `map` | Quản lý render địa hình, background, phá hủy map (MM). | `createMap(mapID): void`<br>`createBackGround(): void`<br>`clearBackGround(): void`<br>`getDataByID(id): byte[]`<br>`isTileDestroy(id): boolean`<br>*(+17 methods khác)* |
| **`MapFile`**<br>`map/MapFile.java` | `map` | Quản lý render địa hình, background, phá hủy map (MapFile). | `init(): void` |
| **`MapImage`**<br>`map/MapImage.java` | `map` | Quản lý render địa hình, background, phá hủy map (MapImage). | - |
| **`AvMain`**<br>`model/AvMain.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (AvMain). | `paint(g): void`<br>`paintLoadImg(g, f, xp, yp, anchor): void`<br>`update(): void`<br>`keypress(keyCode): void`<br>`paintCmd(g): void`<br>*(+40 methods khác)* |
| **`AvatarInfo`**<br>`model/AvatarInfo.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (AvatarInfo). | - |
| **`BoardInfo`**<br>`model/BoardInfo.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (BoardInfo). | - |
| **`CRes`**<br>`model/CRes.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (CRes). | `init(): void`<br>`sin(a): int`<br>`cos(a): int`<br>`tan(a): int`<br>`atan(a): int`<br>*(+51 methods khác)* |
| **`CTime`**<br>`model/CTime.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (CTime). | `initTimeInterval(time): void`<br>`resetTime(): void`<br>`skipTurn(): void`<br>`update(): void`<br>`paint(g): void`<br>*(+1 methods khác)* |
| **`ChatPopup`**<br>`model/ChatPopup.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (ChatPopup). | `show(time, x, y, chat): void`<br>`showSingle(time, x, y, chat): void`<br>`update(): void`<br>`paint(g): void` |
| **`Clan`**<br>`model/Clan.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Clan). | - |
| **`ClanItem`**<br>`model/ClanItem.java` | `model` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | - |
| **`Dialog`**<br>`model/Dialog.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Dialog). | `update(): void`<br>`paint(g): void`<br>`keyPress(keyCode): void`<br>`onPointerPressed(x, y, index): void`<br>`onPointerReleased(x, y, index): void`<br>*(+1 methods khác)* |
| **`FilePack`**<br>`model/FilePack.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (FilePack). | `loadFile(fileName): byte[]`<br>`loadImage(fileName): mImage`<br>`loadImage(fileName, action2): mImage`<br>`loadImage(fileName, isFont): mImage`<br>`LoadAllNameInFile(): void`<br>*(+3 methods khác)* |
| **`Fomula`**<br>`model/Fomula.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Fomula). | `getImageMaterial(id): mImage`<br>`paint(g): void`<br>`update(): void`<br>`input(): void` |
| **`Font`**<br>`model/Font.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Font). | `perform(object): void`<br>`drawString(g, st, x, y, align, isClip): void`<br>`drawString(g, st, x, y, align): void`<br>`getWidth(st): int`<br>`replace(_text, _searchStr, _replacementStr): String`<br>*(+3 methods khác)* |
| **`FrameImage`**<br>`model/FrameImage.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (FrameImage). | `perform(object): void`<br>`drawRegionFrame(idx, x, y, trans, anchor, g, bulletID, n): void`<br>`drawFrame(idx, x, y, trans, anchor, g): void`<br>`fillFrame(idx, x, y, percentFill, trans, anchor, g, isClip): void`<br>`drawFrame(idx, x, y, trans, anchor, g, isClip): void`<br>*(+1 methods khác)* |
| **`GetString`**<br>`model/GetString.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (GetString). | - |
| **`Gift`**<br>`model/Gift.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Gift). | - |
| **`IAction`**<br>`model/IAction.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (IAction). | - |
| **`IAction2`**<br>`model/IAction2.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (IAction2). | - |
| **`IActionListener`**<br>`model/IActionListener.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (IActionListener). | - |
| **`IconManager`**<br>`model/IconManager.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (IconManager). | `addIcon(clan): void`<br>`isExist(clanID): boolean`<br>`getImage(clanID): Image` |
| **`ImageIcon`**<br>`model/ImageIcon.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (ImageIcon). | - |
| **`InfoPopup`**<br>`model/InfoPopup.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (InfoPopup). | `gI(): InfoPopup`<br>`show(): void`<br>`setInfo(info): void`<br>`update(): void`<br>`paint(g): void` |
| **`InputDlg`**<br>`model/InputDlg.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (InputDlg). | `setInfo(info, ok, back, type): void`<br>`update(): void`<br>`paint(g): void`<br>`keyPress(keyCode): void`<br>`onPointerPressed(x, y2, index): void`<br>*(+2 methods khác)* |
| **`InviteInfo`**<br>`model/InviteInfo.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (InviteInfo). | - |
| **`KeyConstant`**<br>`model/KeyConstant.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (KeyConstant). | - |
| **`Language`**<br>`model/Language.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Language). | `backVersion(): String`<br>`reConnect(): String`<br>`createCharName(): String`<br>`dangkyFail(): String`<br>`dangkySucceed(): String`<br>*(+320 methods khác)* |
| **`LevelDetail`**<br>`model/LevelDetail.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (LevelDetail). | - |
| **`LuckyGift`**<br>`model/LuckyGift.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (LuckyGift). | `setIcon(dataRaw, len): void`<br>`paint(g, x, y): void`<br>`update(): void` |
| **`MaterialIconMn`**<br>`model/MaterialIconMn.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (MaterialIconMn). | `addIcon(img): void`<br>`isExistIcon(id): boolean`<br>`getImageFromID(id): mImage` |
| **`Menu`**<br>`model/Menu.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Menu). | `startAt(menuItems, pos): void`<br>`moveCamera(): void`<br>`onPointerPressed(x, y, index): void`<br>`onPointerReleased(x, y, index): void`<br>`onPointerDragged(x, y, index): void`<br>*(+5 methods khác)* |
| **`Mission`**<br>`model/Mission.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Mission). | - |
| **`MoneyInfo`**<br>`model/MoneyInfo.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (MoneyInfo). | - |
| **`MsgDlg`**<br>`model/MsgDlg.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (MsgDlg). | `show(): void`<br>`close(): void`<br>`setInfo(info, left, center, right): void`<br>`setInfo(info, timeShow, action, left, center, right): void`<br>`setInfo(info, timeShow, action): void`<br>*(+4 methods khác)* |
| **`MsgInfo`**<br>`model/MsgInfo.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (MsgInfo). | - |
| **`MsgPopup`**<br>`model/MsgPopup.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (MsgPopup). | `paint(g): void`<br>`onPointerDragged(xDrag, yDrag, index): void`<br>`onPointerPressed(xScreen, yScreen, index): void`<br>`onPointerReleased(xRealse, yRealse, index): void` |
| **`PauseMenu`**<br>`model/PauseMenu.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (PauseMenu). | `startAt(menuItems): void`<br>`onPointerPressed(xScreen, yScreen, index): void`<br>`onPointerDrag(xScreen, yScreen, index): void`<br>`onPointerRealeased(xRealeased, yRealeased, index): void`<br>`paint(g): void`<br>*(+1 methods khác)* |
| **`PlayerInfo`**<br>`model/PlayerInfo.java` | `model` | Logic, trạng thái, hoạt ảnh nhân vật/boss (PlayerInfo). | `getAttribute(): void`<br>`setAllEquipEffect(): void`<br>`getMyEquip(index): void`<br>`getVipEquip(): void`<br>`getStrMoney(): String`<br>*(+8 methods khác)* |
| **`Popup`**<br>`model/Popup.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Popup). | `paint(g): void`<br>`update(): void`<br>`show(): void`<br>`showSingle(): void`<br>`keyPress(keyCode): void`<br>*(+4 methods khác)* |
| **`Position`**<br>`model/Position.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (Position). | - |
| **`RoomInfo`**<br>`model/RoomInfo.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (RoomInfo). | `getStat(): void` |
| **`SMS`**<br>`model/SMS.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (SMS). | - |
| **`TField`**<br>`model/TField.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (TField). | `perform(): void`<br>`doChangeToTextBox(): void`<br>`setisFocus(isFocus): void`<br>`setisVisible(isVisible): void`<br>`setHint(utf): void`<br>*(+19 methods khác)* |
| **`TimeBomb`**<br>`model/TimeBomb.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (TimeBomb). | `paint(g): void`<br>`update(): void` |
| **`UserData`**<br>`model/UserData.java` | `model` | Dữ liệu thực thể, font, âm thanh hoặc dialog phụ trợ (UserData). | - |
| **`Cmd_Client2Server`**<br>`network/Cmd_Client2Server.java` | `network` | Giao thức mạng, gửi/nhận message client-server (Cmd_Client2Server). | - |
| **`Cmd_Server2Client`**<br>`network/Cmd_Server2Client.java` | `network` | Giao thức mạng, gửi/nhận message client-server (Cmd_Server2Client). | - |
| **`Command`**<br>`network/Command.java` | `network` | Giao thức mạng, gửi/nhận message client-server (Command). | `paint(g): void` |
| **`GameLogicHandler`**<br>`network/GameLogicHandler.java` | `network` | Giao thức mạng, gửi/nhận message client-server (GameLogicHandler). | `gI(): GameLogicHandler`<br>`loadRMS(filename): byte[]`<br>`saveRMS(filename, data): void`<br>`saveIP(strID): void`<br>`loadIP(): String`<br>*(+85 methods khác)* |
| **`GameService`**<br>`network/GameService.java` | `network` | Giao thức mạng, gửi/nhận message client-server (GameService). | `setSession(gi): void`<br>`gI(): GameService`<br>`login(username, pass, version): void`<br>`requestRoomList(): void`<br>`requestBoardList(id): void`<br>*(+90 methods khác)* |
| **`IGameLogicHandler`**<br>`network/IGameLogicHandler.java` | `network` | Giao thức mạng, gửi/nhận message client-server (IGameLogicHandler). | - |
| **`IMessageHandler`**<br>`network/IMessageHandler.java` | `network` | Giao thức mạng, gửi/nhận message client-server (IMessageHandler). | - |
| **`ISession`**<br>`network/ISession.java` | `network` | Giao thức mạng, gửi/nhận message client-server (ISession). | - |
| **`Message`**<br>`network/Message.java` | `network` | Giao thức mạng, gửi/nhận message client-server (Message). | `getData(): byte[]`<br>`reader(): DataInputStream`<br>`writer(): DataOutputStream`<br>`cleanup(): void` |
| **`MessageHandler`**<br>`network/MessageHandler.java` | `network` | Giao thức mạng, gửi/nhận message client-server (MessageHandler). | `gI(): MessageHandler`<br>`onConnectOK(): void`<br>`onConnectionFail(): void`<br>`onDisconnected(): void`<br>`onMessage(msg): void`<br>*(+14 methods khác)* |
| **`Session_ME`**<br>`network/Session_ME.java` | `network` | Giao thức mạng, gửi/nhận message client-server (Session_ME). | `gI(): Session_ME`<br>`isConnected(): boolean`<br>`setHandler(messageHandler): void`<br>`connect(host, port): void`<br>`onRecieveMsg(msg): void`<br>*(+12 methods khác)* |
| **`Boss`**<br>`player/Boss.java` | `player` | Logic, trạng thái, hoạt ảnh nhân vật/boss (Boss). | `onPointerPressed(x, y, index): void`<br>`paintBigRobot(g): void`<br>`paintKhiCau(g): void`<br>`checkFrameBalloonGun(): void`<br>`paintFan1(g): void`<br>*(+13 methods khác)* |
| **`CPlayer`**<br>`player/CPlayer.java` | `player` | Logic, trạng thái, hoạt ảnh nhân vật/boss (CPlayer). | `init(): void`<br>`capture(whoCapture): void`<br>`ghostHit(player): void`<br>`capturePlayer(): void`<br>`checkMapDeep(deep, x, y): boolean`<br>*(+53 methods khác)* |
| **`PM`**<br>`player/PM.java` | `player` | Logic, trạng thái, hoạt ảnh nhân vật/boss (PM). | `init(): void`<br>`insertPlayer(x, y, index, player, currHP): void`<br>`initPlayer(pX, pY, maxHP): void`<br>`getPlayerNameFromID(id): String`<br>`getPlayerFromID(id): CPlayer`<br>*(+26 methods khác)* |
| **`ArchivementScr`**<br>`screen/ArchivementScr.java` | `screen` | Màn hình UI / Screen giao diện game (ArchivementScr). | `perform(): void`<br>`paint(g): void`<br>`update(): void` |
| **`BoardListScr`**<br>`screen/BoardListScr.java` | `screen` | Màn hình UI / Screen giao diện game (BoardListScr). | `moveCamera(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>`setBoardName(id, name): void`<br>*(+8 methods khác)* |
| **`CScreen`**<br>`screen/CScreen.java` | `screen` | Màn hình UI / Screen giao diện game (CScreen). | `paint(_mGraphic): void`<br>`paintDefaultBg(g): void`<br>`paintLevelUp(g, x): void`<br>`update(): void`<br>`mainLoop(): void`<br>*(+26 methods khác)* |
| **`ChangePlayerCSr`**<br>`screen/ChangePlayerCSr.java` | `screen` | Logic, trạng thái, hoạt ảnh nhân vật/boss (ChangePlayerCSr). | `perform(): void`<br>`doClose(): void`<br>`doChangePlayer(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+17 methods khác)* |
| **`ClanScreen`**<br>`screen/ClanScreen.java` | `screen` | Màn hình UI / Screen giao diện game (ClanScreen). | `transTextLimit(pos, limit): Position`<br>`moveCamera(): void`<br>`doDetail(): void`<br>`perform(): void`<br>`getClanList(page, clan): void`<br>*(+27 methods khác)* |
| **`ConfigScr`**<br>`screen/ConfigScr.java` | `screen` | Màn hình UI / Screen giao diện game (ConfigScr). | `perform(): void`<br>`show(): void`<br>`paint(g): void`<br>`onPointerDragged(xDrag, yDrag, index): void`<br>`onPointerPressed(xScreen, yScreen, index): void`<br>*(+2 methods khác)* |
| **`EquipScreen`**<br>`screen/EquipScreen.java` | `screen` | Màn hình UI / Screen giao diện game (EquipScreen). | `perform(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+43 methods khác)* |
| **`FomulaScreen`**<br>`screen/FomulaScreen.java` | `screen` | Màn hình UI / Screen giao diện game (FomulaScreen). | `setFomula(f): void`<br>`show(): void`<br>`show(last): void`<br>`moveCamera(): void`<br>`input(): void`<br>*(+8 methods khác)* |
| **`GameScr`**<br>`screen/GameScr.java` | `screen` | Màn hình UI / Screen giao diện game (GameScr). | `perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>*(+97 methods khác)* |
| **`Inventory`**<br>`screen/Inventory.java` | `screen` | Màn hình UI / Screen giao diện game (Inventory). | `perform(): void`<br>`show(lastScreen): void`<br>`init(): void`<br>`getCommand(): void`<br>`perform(): void`<br>*(+28 methods khác)* |
| **`LevelScreen`**<br>`screen/LevelScreen.java` | `screen` | Màn hình UI / Screen giao diện game (LevelScreen). | `show(lastScr): void`<br>`doClose(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+11 methods khác)* |
| **`ListScr`**<br>`screen/ListScr.java` | `screen` | Màn hình UI / Screen giao diện game (ListScr). | `getPlayerIcon(clanID, icon): void`<br>`moveCamera(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+28 methods khác)* |
| **`LoginScr`**<br>`screen/LoginScr.java` | `screen` | Màn hình UI / Screen giao diện game (LoginScr). | `show(): void`<br>`getUrlUpdateGame(): String`<br>`connect(): void`<br>`doForgetPass(user): void`<br>`perform(): void`<br>*(+40 methods khác)* |
| **`LuckyGame`**<br>`screen/LuckyGame.java` | `screen` | Màn hình UI / Screen giao diện game (LuckyGame). | `perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(object): void`<br>`perform(): void`<br>*(+25 methods khác)* |
| **`LuckyGifrScreen`**<br>`screen/LuckyGifrScreen.java` | `screen` | Màn hình UI / Screen giao diện game (LuckyGifrScreen). | `show(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>`paint(g): void`<br>*(+6 methods khác)* |
| **`MenuScr`**<br>`screen/MenuScr.java` | `screen` | Màn hình UI / Screen giao diện game (MenuScr). | `getIdMenu(type): void`<br>`goToGame(): void`<br>`transTextLimit(pos, limit): Position`<br>`getRectHeight(): void`<br>`show(): void`<br>*(+44 methods khác)* |
| **`MissionScreen`**<br>`screen/MissionScreen.java` | `screen` | Màn hình UI / Screen giao diện game (MissionScreen). | `perform(): void`<br>`show(): void`<br>`getCurrMiss(): Mission`<br>`paintList(g): void`<br>`paint(g): void`<br>*(+9 methods khác)* |
| **`MoneyScr`**<br>`screen/MoneyScr.java` | `screen` | Màn hình UI / Screen giao diện game (MoneyScr). | `isHaveMoneyList(): boolean`<br>`priceFromID(id): int`<br>`startAnimLeft(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+22 methods khác)* |
| **`MoneyScrIOS`**<br>`screen/MoneyScrIOS.java` | `screen` | Màn hình UI / Screen giao diện game (MoneyScrIOS). | `priceFromID(id): int`<br>`startAnimLeft(): void`<br>`perform(): void`<br>`perform(): void`<br>`doBuy(): void`<br>*(+11 methods khác)* |
| **`MsgScreen`**<br>`screen/MsgScreen.java` | `screen` | Màn hình UI / Screen giao diện game (MsgScreen). | `show(lastScr): void`<br>`moveCamera(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+24 methods khác)* |
| **`PrepareScr`**<br>`screen/PrepareScr.java` | `screen` | Màn hình UI / Screen giao diện game (PrepareScr). | `init(): void`<br>`getPlayerIcon(clanID, icon): void`<br>`initItemCurrent(): void`<br>`getStrainingItem(): int[]`<br>`copyItemCurrent(): int[]`<br>*(+85 methods khác)* |
| **`QuangCao`**<br>`screen/QuangCao.java` | `screen` | Màn hình UI / Screen giao diện game (QuangCao). | `getCommand(): void`<br>`perform(): void`<br>`perform(): void`<br>`paint(g): void` |
| **`RoomListScr`**<br>`screen/RoomListScr.java` | `screen` | Màn hình UI / Screen giao diện game (RoomListScr). | `moveCamera(): void`<br>`init(): void`<br>`perform(): void`<br>`perform(): void`<br>`show(): void`<br>*(+9 methods khác)* |
| **`RoomListScr2`**<br>`screen/RoomListScr2.java` | `screen` | Màn hình UI / Screen giao diện game (RoomListScr2). | `gI(): RoomListScr`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+18 methods khác)* |
| **`ServerListScreen`**<br>`screen/ServerListScreen.java` | `screen` | Màn hình UI / Screen giao diện game (ServerListScreen). | `perform(): void`<br>`perform(): void`<br>`perform(): void`<br>`paint(g): void`<br>`update(): void`<br>*(+3 methods khác)* |
| **`SplashScr`**<br>`screen/SplashScr.java` | `screen` | Màn hình UI / Screen giao diện game (SplashScr). | `paint(g): void`<br>`show(): void`<br>`update(): void` |
| **`TabScreen`**<br>`screen/TabScreen.java` | `screen` | Màn hình UI / Screen giao diện game (TabScreen). | `getW(): void`<br>`paint(g): void`<br>`paintSuper(g): void`<br>`update(): void`<br>`show(lastScreen): void` |
| **`ShopBietDoi`**<br>`shop/ShopBietDoi.java` | `shop` | Giao diện & xử lý cửa hàng mua bán (ShopBietDoi). | `getCurrItem(): ClanItem`<br>`getClanItem(id): ClanItem`<br>`initCommand(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+14 methods khác)* |
| **`ShopEquipment`**<br>`shop/ShopEquipment.java` | `shop` | Quản lý model, dữ liệu và render trang bị (ShopEquipment). | `perform(): void`<br>`show(lastScreen): void`<br>`paint(g): void`<br>`update(): void`<br>`mainLoop(): void`<br>*(+15 methods khác)* |
| **`ShopItem`**<br>`shop/ShopItem.java` | `shop` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `show(lastScreen): void`<br>`getCommand(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+27 methods khác)* |
| **`ShopLinhTinh`**<br>`shop/ShopLinhTinh.java` | `shop` | Giao diện & xử lý cửa hàng mua bán (ShopLinhTinh). | `getCommand(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>`perform(): void`<br>*(+24 methods khác)* |

## 5. Chi tiết Server Classes (91 files)

| File / Class | Package | Mục đích | Các Method chính |
|---|---|---|---|
| **`Cache`**<br>`mobiarmy/Cache.java` | `mobiarmy` | Module chức năng Cache. | `main(args): void` |
| **`MobiArmy`**<br>`mobiarmy/MobiArmy.java` | `mobiarmy` | Module chức năng MobiArmy. | `main(args): void` |
| **`Test`**<br>`mobiarmy/Test.java` | `mobiarmy` | Module chức năng Test. | `paintComponent(g): void`<br>`main(args): void` |
| **`Util`**<br>`mobiarmy/Util.java` | `mobiarmy` | Module chức năng Util. | `nextInt(min, max): int`<br>`nextDouble(): double`<br>`nextInt(bound): int`<br>`formatNum(num): String`<br>`getRandomCharacters(inputString, numberOfChars): String` |
| **`IMessageHandler`**<br>`mobiarmy/io/IMessageHandler.java` | `mobiarmy.io` | Tầng socket I/O, đóng gói và đọc Message nhị phân. | - |
| **`Message`**<br>`mobiarmy/io/Message.java` | `mobiarmy.io` | Tầng socket I/O, đóng gói và đọc Message nhị phân. | `getCommand(): byte`<br>`setCommand(cmd): void`<br>`setCommand(cmd): void`<br>`getData(): byte[]`<br>`reader(): DataInputStream`<br>*(+2 methods khác)* |
| **`Write`**<br>`mobiarmy/io/Write.java` | `mobiarmy.io` | Tầng socket I/O, đóng gói và đọc Message nhị phân. | `writer(): DataOutputStream`<br>`getBytes(): byte[]`<br>`size(): int` |
| **`Bot`**<br>`mobiarmy/server/Bot.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `update(): void`<br>`remove(): void`<br>`addBot(name, glassID, exp, equipID): Bot`<br>`findById(id): Bot`<br>`add(bot): void`<br>*(+2 methods khác)* |
| **`Caption`**<br>`mobiarmy/server/Caption.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `loadCaption(): void` |
| **`Confirm`**<br>`mobiarmy/server/Confirm.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `addConfirm(action, str, object): void`<br>`addConfirm2(action, str, object): void`<br>`confirm(): void` |
| **`ControlHandler`**<br>`mobiarmy/server/ControlHandler.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `handleControlMessage(msg): void` |
| **`DBManager`**<br>`mobiarmy/server/DBManager.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `selectColumnIndex(sql, params): ArrayList<Object[]>`<br>`selectColumnName(sql, params): ArrayList<DataRow>`<br>`update(sql, params): int`<br>`updateWithMap(sql, HashMap<String, values, params): int`<br>`insertWithMap(tableName, HashMap<String, values): int`<br>*(+7 methods khác)* |
| **`Equip`**<br>`mobiarmy/server/Equip.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `slot(): int`<br>`priceSlot(): int`<br>`calculatePrice(): int`<br>`renewalPrice(): int`<br>`date(): int`<br>*(+4 methods khác)* |
| **`Exp`**<br>`mobiarmy/server/Exp.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `loadExp(): void`<br>`getLevelExp(exp): int`<br>`get(level): Exp`<br>`getPercentExp(level, exp): int` |
| **`GameData`**<br>`mobiarmy/server/GameData.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `loadHole(): void`<br>`getCache(url): byte[]`<br>`loadMapIcon(): void`<br>`loadLayer(): void`<br>`getHoleType(bulletType): byte`<br>*(+5 methods khác)* |
| **`Glass`**<br>`mobiarmy/server/Glass.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `loadGlass(): void`<br>`updateAll(): void`<br>`getPercentLevel(): int`<br>`getExpMaxLevel(): int`<br>`upadtePoint(pointAdd): void`<br>*(+3 methods khác)* |
| **`Item`**<br>`mobiarmy/server/Item.java` | `mobiarmy.server` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `loadItem(): void`<br>`get(id): Item`<br>`deepCopy(): Item` |
| **`LinhTinh`**<br>`mobiarmy/server/LinhTinh.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `loadLinhTinh(): void`<br>`get(id): LinhTinh`<br>`deepCopy(): LinhTinh` |
| **`Map`**<br>`mobiarmy/server/Map.java` | `mobiarmy.server` | Quản lý render địa hình, background, phá hủy map (Map). | `loadMap(): void`<br>`get(mapID): Map` |
| **`MapBoss`**<br>`mobiarmy/server/MapBoss.java` | `mobiarmy.server` | Quản lý render địa hình, background, phá hủy map (MapBoss). | `loadMapBoss(): void` |
| **`MessageHandler`**<br>`mobiarmy/server/MessageHandler.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `startReadingThread(): void`<br>`startSendingThread(): void`<br>`close(): void`<br>`doSendMessage(message): void`<br>`setKey(): void`<br>*(+2 methods khác)* |
| **`Mission`**<br>`mobiarmy/server/Mission.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `loadMession(): void`<br>`get(id, level): Mission`<br>`deepCopy(): Mission` |
| **`Pack`**<br>`mobiarmy/server/Pack.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `xorEncrypt(data): byte[]`<br>`addEntry(name, data): void`<br>`toByteArray(): byte[]` |
| **`Select`**<br>`mobiarmy/server/Select.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `reset(): void`<br>`addElement(type, id, quntity): void`<br>`sizeOfType(type): int`<br>`getFirstOfType(type): Element`<br>`make(): void` |
| **`Server`**<br>`mobiarmy/server/Server.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `start(): void`<br>`stop(): void`<br>`isRunning(): boolean`<br>`addCommand(session, message): void`<br>`addCommand(cmd, object): void`<br>*(+1 methods khác)* |
| **`ServerManager`**<br>`mobiarmy/server/ServerManager.java` | `default` | Xử lý session người chơi, database, handler gói tin và quản lý server. | - |
| **`Session`**<br>`mobiarmy/server/Session.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `sendMessage(message): void`<br>`requestDisconnect(): void`<br>`disconnect(): void`<br>`isGE(compareVersion): boolean`<br>`start(): void` |
| **`SessionHandler`**<br>`mobiarmy/server/SessionHandler.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `log(str): void`<br>`loadInfoAll(): void`<br>`layerData(): void`<br>`topInfo(): void`<br>`iconInfo(version, version2): void`<br>*(+76 methods khác)* |
| **`SessionManager`**<br>`mobiarmy/server/SessionManager.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `addSession(session): void`<br>`addUser(user): void`<br>`generateUsers(): ArrayList<User>`<br>`removeSession(session): void`<br>`removeUserById(id): void`<br>*(+10 methods khác)* |
| **`ShopEquipment`**<br>`mobiarmy/server/ShopEquipment.java` | `mobiarmy.server` | Quản lý model, dữ liệu và render trang bị (ShopEquipment). | `loadShopEquipment(): void`<br>`generate(glassID): ArrayList<Equip>` |
| **`ShopLinhTinh`**<br>`mobiarmy/server/ShopLinhTinh.java` | `mobiarmy.server` | Giao diện & xử lý cửa hàng mua bán (ShopLinhTinh). | `loadShopLinhTinh(): void`<br>`get(id): LinhTinh` |
| **`Text`**<br>`mobiarmy/server/Text.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `__(string): String` |
| **`User`**<br>`mobiarmy/server/User.java` | `mobiarmy.server` | Xử lý session người chơi, database, handler gói tin và quản lý server. | `getEquips(): Equip[]`<br>`connect(session): boolean`<br>`glass(): Glass`<br>`addXu(add, flag): void`<br>`addCup(add, flag): void`<br>*(+55 methods khác)* |
| **`Gun`**<br>`mobiarmy/war/Gun.java` | `mobiarmy.war` | Động cơ chiến đấu (battle engine), phòng đấu, vật lý và xử lý lượt bắn. | `shootBullet(bulletId, x, y, width, height, ang, force, force2, nshoot, att, radius): void`<br>`fillXY(): void` |
| **`MapData`**<br>`mobiarmy/war/MapData.java` | `mobiarmy.war` | Quản lý render địa hình, background, phá hủy map (MapData). | `startGame(users): void`<br>`setData(data, W, H): void`<br>`inRegion(x, y, x0, y0, w, h): boolean`<br>`isNotAlpha(rgb): boolean`<br>`isCollision(x, y): boolean`<br>*(+52 methods khác)* |
| **`PathSimulator`**<br>`mobiarmy/war/PathSimulator.java` | `mobiarmy.war` | Động cơ chiến đấu (battle engine), phòng đấu, vật lý và xử lý lượt bắn. | `simulate(): boolean` |
| **`Player`**<br>`mobiarmy/war/Player.java` | `mobiarmy.war` | Logic, trạng thái, hoạt ảnh nhân vật/boss (Player). | `bulletIdByGlassID(): byte`<br>`updateHP(addHP): void`<br>`updateAngry(addAngry): void`<br>`die(): void`<br>`chuanHoaXY(): void`<br>*(+11 methods khác)* |
| **`Poison`**<br>`mobiarmy/war/Poison.java` | `mobiarmy.war` | Động cơ chiến đấu (battle engine), phòng đấu, vật lý và xử lý lượt bắn. | - |
| **`Room`**<br>`mobiarmy/war/Room.java` | `mobiarmy.war` | Động cơ chiến đấu (battle engine), phòng đấu, vật lý và xử lý lượt bắn. | `loadRoom(): void` |
| **`RoomInfo`**<br>`mobiarmy/war/RoomInfo.java` | `mobiarmy.war` | Động cơ chiến đấu (battle engine), phòng đấu, vật lý và xử lý lượt bắn. | `loadRoomInfo(): void`<br>`get(id): RoomInfo`<br>`get(roomID, boardID): RoomWait`<br>`update(): void` |
| **`RoomWait`**<br>`mobiarmy/war/RoomWait.java` | `mobiarmy.war` | Động cơ chiến đấu (battle engine), phòng đấu, vật lý và xử lý lượt bắn. | `reset(): void`<br>`join(user, pass): void`<br>`nextOwner(): void`<br>`generateIndex(): byte`<br>`getUser(userID): User`<br>*(+15 methods khác)* |
| **`TimeBomb`**<br>`mobiarmy/war/TimeBomb.java` | `mobiarmy.war` | Động cơ chiến đấu (battle engine), phòng đấu, vật lý và xử lý lượt bắn. | `update(): void` |
| **`Tornado`**<br>`mobiarmy/war/Tornado.java` | `mobiarmy.war` | Động cơ chiến đấu (battle engine), phòng đấu, vật lý và xử lý lượt bắn. | `update(): void` |
| **`BigBoom`**<br>`mobiarmy/war/Boss/BigBoom.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (BigBoom) trong trận chiến. | `update(): void` |
| **`Boss`**<br>`mobiarmy/war/Boss/Boss.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (Boss) trong trận chiến. | `getCommand(id): Command`<br>`addCommand(id, objects): void`<br>`updateTurn(): void`<br>`update(): void` |
| **`GiftBox`**<br>`mobiarmy/war/Boss/GiftBox.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (GiftBox) trong trận chiến. | - |
| **`GiftBox2`**<br>`mobiarmy/war/Boss/GiftBox2.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (GiftBox2) trong trận chiến. | - |
| **`Robot`**<br>`mobiarmy/war/Boss/Robot.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (Robot) trong trận chiến. | `update(): void` |
| **`RobotSpider`**<br>`mobiarmy/war/Boss/RobotSpider.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (RobotSpider) trong trận chiến. | `update(): void` |
| **`SmallBoom`**<br>`mobiarmy/war/Boss/SmallBoom.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (SmallBoom) trong trận chiến. | `update(): void` |
| **`Trex`**<br>`mobiarmy/war/Boss/Trex.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (Trex) trong trận chiến. | `update(): void` |
| **`Ufo`**<br>`mobiarmy/war/Boss/Ufo.java` | `mobiarmy.war.Boss` | AI và logic hành vi của Boss (Ufo) trong trận chiến. | `update(): void`<br>`die(): void` |
| **`AddBoomBullet`**<br>`mobiarmy/war/Boss/bullet/AddBoomBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`ApacheBullet`**<br>`mobiarmy/war/Boss/bullet/ApacheBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`BicycleBullet`**<br>`mobiarmy/war/Boss/bullet/BicycleBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`BigBoomBullet`**<br>`mobiarmy/war/Boss/bullet/BigBoomBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`BigLaserBullet`**<br>`mobiarmy/war/Boss/bullet/BigLaserBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`BigRocketBullet`**<br>`mobiarmy/war/Boss/bullet/BigRocketBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`BomBullet`**<br>`mobiarmy/war/Boss/bullet/BomBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`Bullet`**<br>`mobiarmy/war/Boss/bullet/Bullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `collision(): void`<br>`nextXY(): void`<br>`getCollision(X1, Y1, X2, Y2): int[]` |
| **`BulletTrajectory`**<br>`mobiarmy/war/Boss/bullet/BulletTrajectory.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `run(): void`<br>`start(): void`<br>`calculateTrajectory(): void` |
| **`ChickyBullet`**<br>`mobiarmy/war/Boss/bullet/ChickyBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`FlyPortBullet`**<br>`mobiarmy/war/Boss/bullet/FlyPortBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`ItemB52Bullet`**<br>`mobiarmy/war/Boss/bullet/ItemB52Bullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemBlindBullet`**<br>`mobiarmy/war/Boss/bullet/ItemBlindBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemBomBullet`**<br>`mobiarmy/war/Boss/bullet/ItemBomBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemEarthHole2Bullet`**<br>`mobiarmy/war/Boss/bullet/ItemEarthHole2Bullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemEarthHoleBullet`**<br>`mobiarmy/war/Boss/bullet/ItemEarthHoleBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemFreezeBullet`**<br>`mobiarmy/war/Boss/bullet/ItemFreezeBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemLandBullet`**<br>`mobiarmy/war/Boss/bullet/ItemLandBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemLaserBullet`**<br>`mobiarmy/war/Boss/bullet/ItemLaserBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemMeteorBullet`**<br>`mobiarmy/war/Boss/bullet/ItemMeteorBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemMouseBullet`**<br>`mobiarmy/war/Boss/bullet/ItemMouseBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemNoCollisionMap`**<br>`mobiarmy/war/Boss/bullet/ItemNoCollisionMap.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemPoisonBullet`**<br>`mobiarmy/war/Boss/bullet/ItemPoisonBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemRainBullet`**<br>`mobiarmy/war/Boss/bullet/ItemRainBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemRocketBullet`**<br>`mobiarmy/war/Boss/bullet/ItemRocketBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemSilkBullet`**<br>`mobiarmy/war/Boss/bullet/ItemSilkBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemSilkBullet2`**<br>`mobiarmy/war/Boss/bullet/ItemSilkBullet2.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemSuicideBullet`**<br>`mobiarmy/war/Boss/bullet/ItemSuicideBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemTeleportBullet`**<br>`mobiarmy/war/Boss/bullet/ItemTeleportBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemTeleportBullet2`**<br>`mobiarmy/war/Boss/bullet/ItemTeleportBullet2.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemTimeBombBullet`**<br>`mobiarmy/war/Boss/bullet/ItemTimeBombBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemTornadoBullet`**<br>`mobiarmy/war/Boss/bullet/ItemTornadoBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`ItemUfo`**<br>`mobiarmy/war/Boss/bullet/ItemUfo.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `update(): void`<br>`die(): void` |
| **`ItemUfoBullet`**<br>`mobiarmy/war/Boss/bullet/ItemUfoBullet.java` | `mobiarmy.war.Boss.bullet` | Vật phẩm và đường đạn (đạn, bom, item hỗ trợ) trong trận đấu. | `nextXY(): void` |
| **`MirrorBullet`**<br>`mobiarmy/war/Boss/bullet/MirrorBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`RocketExpBullet`**<br>`mobiarmy/war/Boss/bullet/RocketExpBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`SmallBoomBullet`**<br>`mobiarmy/war/Boss/bullet/SmallBoomBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`TarzanBullet`**<br>`mobiarmy/war/Boss/bullet/TarzanBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void` |
| **`UfoLaserBullet`**<br>`mobiarmy/war/Boss/bullet/UfoLaserBullet.java` | `mobiarmy.war.Boss.bullet` | Đạn đặc thù hoặc đạn boss: tính quỹ đạo bay, va chạm & sát thương. | `nextXY(): void`<br>`isCollisionPlayer(x, y): boolean`<br>`getCollision(X1, Y1, X2, Y2): int[]` |

## 6. Sổ tay tra cứu Bug thường gặp (AI Debug Guide)
1. **Lỗi hiển thị/flicker nhân vật & đồ**: Xem `Equipment/PlayerEquip.java` (`paintFace`, `paint`), `Equipment/Equip.java` (`drawImage` frame clamp), `player/CPlayer.java` (`paintEquip`).
2. **Lỗi lực bắn / thanh UI**: Xem `screen/GameScr.java` (`onDrawPowerBar`, `maxforce`).
3. **Lỗi vẽ bản đồ nền / đen map**: Xem `map/Background.java` (`nBgX`, `curBGType`) và `map/CMap.java`.
4. **Lỗi thoát trận / Luyện tập / Không chuyển màn**: Đối chiếu `screen/GameScr.java` (sự kiện click khi kết thúc/LEAVEBATTLE) và `network/MessageHandler.java` (case 83 / 20) kết hợp với Server `server/MessageHandler.java` (training ack).
5. **Lỗi trang bị / Click nhầm item / Double click**: Xem `screen/EquipScreen.java` (`onPointerPressed`, `onPointerReleased`, `doFire`, `getBaseAttribute`) và `coreLG/CCanvas.java` (`MAX_TIME_CLICK`).