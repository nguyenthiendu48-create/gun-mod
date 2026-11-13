// Mod Súng Thật (AK-47 & AWP) cho EaglerForge
ModAPI.require("player");
ModAPI.require("network");

ModAPI.displayToChat({ msg: "§6[Realistic Guns] §fĐã nạp! §eCuốc Sắt = AK-47 §f| §bCuốc Kim Cương = AWP" });
ModAPI.displayToChat({ msg: "§7Chuột phải: Bắn | Phím R: Nạp đạn | Shift: Ngắm chuẩn" });

// Cấu hình thông số từng loại súng
const GUNS = {
    292: { // Iron Hoe -> AK-47
        name: "AK-47",
        maxAmmo: 30,
        ammo: 30,
        fireRate: 3,      // Tốc độ bắn (càng nhỏ càng nhanh)
        damage: 7,        // Sát thương mỗi viên
        recoil: 1.8,      // Độ giật nòng súng
        reloadTime: 40,   // Thời gian nạp đạn (40 tick = 2 giây)
        soundPitch: 1.4
    },
    293: { // Diamond Hoe -> AWP Sniper
        name: "AWP",
        maxAmmo: 5,
        ammo: 5,
        fireRate: 25,
        damage: 25,       // Một viên hạ gục mục tiêu
        recoil: 5.5,
        reloadTime: 60,
        soundPitch: 0.7
    }
};

let cooldown = 0;
let reloadTimer = 0;
let isReloading = false;

// Lắng nghe phím R (Mã phím 19) để nạp đạn
window.addEventListener("keydown", (e) => {
    if (e.keyCode === 82 && !isReloading) { // Phím R trên bàn phím
        const held = ModAPI.player.getHeldItem();
        if (held && GUNS[held.itemId]) {
            const gun = GUNS[held.itemId];
            if (gun.ammo < gun.maxAmmo) {
                isReloading = true;
                reloadTimer = gun.reloadTime;
                ModAPI.displayToChat({ msg: `§e[${gun.name}] Đang thay băng đạn...` });
                ModAPI.player.playSound({ name: "random.click", volume: 1.0, pitch: 0.8 });
            }
        }
    }
});

// Vòng lặp xử lý bắn và nạp đạn mỗi khung hình
ModAPI.addEventListener("update", () => {
    if (cooldown > 0) cooldown--;

    const held = ModAPI.player.getHeldItem();
    if (!held || !GUNS[held.itemId]) {
        isReloading = false;
        return;
    }

    const gun = GUNS[held.itemId];

    // Xử lý đếm ngược thời gian nạp đạn
    if (isReloading) {
        reloadTimer--;
        if (reloadTimer <= 0) {
            gun.ammo = gun.maxAmmo;
            isReloading = false;
            ModAPI.displayToChat({ msg: `§a[${gun.name}] Đã nạp đầy đạn (${gun.ammo}/${gun.maxAmmo})!` });
            ModAPI.player.playSound({ name: "random.orb", volume: 0.8, pitch: 1.5 });
        }
        return;
    }

    // Xử lý bóp cò (Chuột phải)
    if (ModAPI.minecraft.gameSettings.keyBindUseItem.pressed && cooldown === 0) {
        if (gun.ammo > 0) {
            fireWeapon(gun);
            gun.ammo--;
            cooldown = gun.fireRate;
            ModAPI.displayToChat({ msg: `§f[${gun.name}] Đạn còn lại: §c${gun.ammo}§f/${gun.maxAmmo}` });
        } else {
            ModAPI.displayToChat({ msg: `§c[${gun.name}] Hết đạn! Bấm phím R để nạp lại.` });
            ModAPI.player.playSound({ name: "random.click", volume: 1.0, pitch: 2.0 });
            cooldown = 15;
        }
    }
});

function fireWeapon(gun) {
    // 1. Âm thanh tiếng súng nổ
    ModAPI.player.playSound({
        name: "random.explode",
        volume: 0.8,
        pitch: gun.soundPitch
    });

    // 2. Tạo độ giật nòng súng (hất nhẹ tâm ngắm lên trên)
    const isSneaking = ModAPI.player.isSneaking();
    const actualRecoil = isSneaking ? gun.recoil * 0.4 : gun.recoil; // Ngồi (Shift) giảm 60% độ giật
    ModAPI.player.rotationPitch -= actualRecoil;

    // 3. Bắn đạn tốc độ cao (dùng mũi tên tàng hình sát thương lớn)
    const yaw = ModAPI.player.rotationYaw * (Math.PI / 180);
    const pitch = ModAPI.player.rotationPitch * (Math.PI / 180);
    const speed = 6.0; // Tốc độ đạn bay thẳng tắp

    const motionX = -Math.sin(yaw) * Math.cos(pitch) * speed;
    const motionY = -Math.sin(pitch) * speed;
    const motionZ = Math.cos(yaw) * Math.cos(pitch) * speed;

    ModAPI.network.sendPacketChatMessage({
        message: `/summon Arrow ~ ~1.5 ~ {Motion:[${motionX.toFixed(2)},${motionY.toFixed(2)},${motionZ.toFixed(2)}],damage:${gun.damage},pickup:0}`
    });
}
// Mod Súng Cơ Bản cho EaglerForge
ModAPI.require("player");
ModAPI.require("network");

// Thông báo khi nạp mod thành công
ModAPI.displayToChat({ msg: "§a[Mod Súng] §fĐã kích hoạt! Cầm Gậy (Stick) và bấm Chuột Phải để bắn." });

// Cài đặt thông số súng
const GUN_ITEM_ID = 280; // ID 280 là cây Gậy (Stick). Bạn có thể đổi sang ID vật phẩm khác.
let cooldown = 0;

// Xử lý sự kiện mỗi khung hình (tick) trong game
ModAPI.addEventListener("update", () => {
    if (cooldown > 0) {
        cooldown--;
    }

    // Kiểm tra xem người chơi có đang cầm Gậy trên tay không
    const heldItem = ModAPI.player.getHeldItem();
    if (heldItem && heldItem.itemId === GUN_ITEM_ID) {
        // Kiểm tra người chơi có đang giữ chuột phải (isUsingItem / right click) không
        if (ModAPI.minecraft.gameSettings.keyBindUseItem.pressed && cooldown === 0) {
            shootBullet();
            cooldown = 4; // Tốc độ bắn (số càng nhỏ bắn càng nhanh, 4 tick = 5 phát/giây)
        }
    }
});

// Hàm bắn đạn và tạo hiệu ứng âm thanh
function shootBullet() {
    // Phát âm thanh tiếng nổ nhỏ khi bóp cò
    ModAPI.player.playSound({
        name: "random.explode",
        volume: 0.4,
        pitch: 1.8
    });

    // Gửi lệnh bắn đạn (triệu hồi cầu tuyết bay theo hướng nhìn)
    // Lưu ý: Chức năng này hoạt động tốt nhất trong thế giới Singleplayer có bật Cheat (Allow Cheats)
    const yaw = ModAPI.player.rotationYaw * (Math.PI / 180);
    const pitch = ModAPI.player.rotationPitch * (Math.PI / 180);

    const speed = 3.5; // Tốc độ bay của viên đạn
    const motionX = -Math.sin(yaw) * Math.cos(pitch) * speed;
    const motionY = -Math.sin(pitch) * speed;
    const motionZ = Math.cos(yaw) * Math.cos(pitch) * speed;

    // Gửi thông báo hoặc chạy lệnh triệu hồi đạn
    ModAPI.network.sendPacketChatMessage({
        message: `/summon Snowball ~ ~1.5 ~ {Motion:[${motionX.toFixed(2)},${motionY.toFixed(2)},${motionZ.toFixed(2)}]}`
    });
}
