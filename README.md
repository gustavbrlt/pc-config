Pour afficher ce README: 
```sh
curl https://raw.githubusercontent.com/gustavbrlt/pc-config/refs/heads/main/README.md
```

Première étape, préparer l'installation:
```sh
cd /etc/nixos
sudo mv hardware-configuration.nix ~
sudo mv configuration.nix ~
sudo chown gustav:users /etc/nixos
nix-shell -p git
```

Puis lancer l'installation (attention: ne fonctionne que si le disque est chiffré):
```sh
sudo git clone https://github.com/gustavbrlt/pc-config.git
sudo mv pc-config/* . && sudo mv pc-config/.git . && sudo mv pc-config/.config . && sudo rm -r pc-config

# 2. On extrait l'UUID dans une variable.
NEW_ROOT=$(sed -n '/fileSystems."\/"/,/}/s/.*by-uuid\/\([^"]*\).*/\1/p' ~/hardware-configuration.nix)

# Si le disque est chiffré :
NEW_BOOT=$(sed -n '/fileSystems."\/boot"/,/}/s/.*by-uuid\/\([^"]*\).*/\1/p' ~/hardware-configuration.nix)
# Pour LUKS, on cherche la ligne qui définit le device
NEW_LUKS=$(sed -n '/boot.initrd.luks.devices/,/;/s/.*by-uuid\/\([^"]*\).*/\1/p' ~/hardware-configuration.nix)

# 3. On applique les changements dans hardware/common.nix
# Remplacement de l'UUID de Root (Système de fichier interne)
sed -i "/fileSystems.\"\/\"/,/}/ s|by-uuid/[^\"]*|by-uuid/$NEW_ROOT|" hardware/common.nix

# Si le disque est chiffré :
# Remplacement de l'UUID de Boot
sed -i "/fileSystems.\"\/boot\"/,/}/ s|by-uuid/[^\"]*|by-uuid/$NEW_BOOT|" hardware/common.nix
# Remplacement de la ligne LUKS complète (Conteneur externe)
# On remplace toute la ligne commençant par boot.initrd.luks pour mettre le bon format et le bon UUID
sed -i "s|^.*boot.initrd.luks.devices.*|  boot.initrd.luks.devices.\"luks-$NEW_LUKS\".device = \"/dev/disk/by-uuid/$NEW_LUKS\";|" hardware/common.nix

# 4. Vérification visuelle
echo "--- Vérification UUID(s) ---"
echo "ROOT: $NEW_ROOT"

# Si le disque est chiffré :
echo "BOOT: $NEW_BOOT"
echo "LUKS: $NEW_LUKS"

grep -E "by-uuid|luks-" hardware/common.nix
sudo nixos-rebuild switch --flake /etc/nixos#default
```
Ensuite s'il n'y a pas eu d'erreur:
```sh
exit
sudo rm ~/configuration.nix ~/hardware-configuration.nix
cd ~
```
