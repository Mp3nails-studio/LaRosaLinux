# La Rosa Live – instrukcja budowy

To jest gotowa konfiguracja **live-build** pod dystrybucję **La Rosa** opartą na Debian Trixie (stable).

## Co jest w środku

- Debian Trixie (stable)
- **XFCE** (lekki, szybki, łatwo brandowalny desktop)
- **Calamares** – prawdziwy graficzny instalator ("zainstaluj na dysk" z poziomu live), z własnym brandingiem La Rosa (logo, tło, kolory panelu bocznego)
- Polski język + klawiatura + Europe/Warsaw
- Motyw: **Arc-Dark** (GTK) + **Papirus-Dark** (ikony) — ciemna, bordowo-różana elegancja floral w stylu lat 2010, dopasowana do dostarczonego logo
- Preinstalowane:
  - Google Chrome (trzeba dodać ręcznie – patrz niżej)
  - Wine + winetricks
  - WPS Office (trzeba dodać ręcznie – patrz niżej)
  - Minetest
  - gnome-software + flatpak
  - Firefox ESR, LibreOffice, VLC, GIMP, Inkscape itd.
- Branding: logo "La Rosa" (róża + skryptowy napis) użyte jako tapeta pulpitu, ikona systemowa, tło i logo instalatora Calamares, motd
- Live ISO (hybrydowe) z opcją instalacji na dysk

## Wymagania sprzętowe do budowy

- Minimum **8 GB RAM** (lepiej 16 GB)
- Minimum **40 GB** wolnego miejsca
- Szybki internet
- Debian lub Ubuntu (najlepiej Debian) jako maszyna budująca

## Przygotowanie

1. Zainstaluj live-build i zależności:

```bash
sudo apt update
sudo apt install live-build debootstrap squashfs-tools xorriso isolinux syslinux-common
```

2. Skopiuj cały katalog `la-rosa` gdzieś na dysk, np.:

```bash
cp -r la-rosa ~/budowanie/
cd ~/budowanie/la-rosa
```

3. **Dodaj Google Chrome i WPS Office** (to najważniejsza ręczna robota — te firmy nie pozwalają redystrybuować swoich .deb przez oficjalne repo Debiana)

### Google Chrome

```bash
mkdir -p config/packages.chroot
cd config/packages.chroot
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
cd ..
```

### WPS Office

Wejdź na https://www.wps.com/office/linux/ i pobierz najnowszy .deb (amd64).
Wrzuć go też do `packages.chroot/`:

```bash
mv ~/Pobrane/wps-office_*.deb config/packages.chroot/
```

## Budowanie na GitHub Actions (bez własnego komputera)

1. Utwórz repozytorium na GitHubie i wrzuć do niego **zawartość** katalogu `la-rosa` (tak, by `auto/`, `config/` i `.github/` były w korzeniu repo).
2. Wejdź w **Actions → Build La Rosa ISO → Run workflow**. Pole z linkiem do WPS Office jest opcjonalne (Chrome pobiera się sam).
3. Po 1–2 godzinach pobierz ISO z sekcji **Artifacts** (`la-rosa-iso`, przechowywane 14 dni). Obok jest plik z sumą SHA256.

Uwaga: w prywatnym repo limit darmowych minut Actions może nie wystarczyć. W publicznym budowanie jest bezpłatne.

## Budowanie lokalne

```bash
chmod +x auto/*
chmod +x config/hooks/live/*
sudo lb clean
sudo lb config
sudo lb build
```

Build zajmie od 30 minut do 2–3 godzin w zależności od maszyny i internetu.

Po zakończeniu dostaniesz plik:

```
live-image-amd64.hybrid.iso
```

Możesz go przepalić na pendrive (balenaEtcher, dd, Ventoy itp.) i uruchomić — na pulpicie live pojawi się ikona **Install La Rosa** (Calamares, z logo i tłem La Rosa), która przeprowadzi Cię przez instalację na dysk krok po kroku.

## Branding — jak to działa

Logo i tapety (dostarczone przez Ciebie, floral bordowy vintage + logo róży) są użyte w pięciu miejscach:

1. **Tapeta pulpitu (domyślna)** — `usr/share/backgrounds/la-rosa/la-rosa-wallpaper.png` — sam floral wzór bez logo, na bordowym tle
2. **Ekran logowania (LightDM) i tło instalatora Calamares** — `la-rosa-wallpaper-login.png` — samo logo wyśrodkowane na czystym czerwonym tle
3. **Tapeta alternatywna** — `la-rosa-wallpaper-alt.png` — floral wzór z logo w centrum; dostępna do wyboru w Ustawieniach → Tło pulpitu (wszystkie trzy warianty są zarejestrowane w `/usr/share/xfce4/backdrops/`, więc pojawią się razem w tym samym miejscu wyboru tapety)
4. **Ikona systemowa** — `usr/share/pixmaps/la-rosa-logo.png` (osobna, kwadratowa wersja logo, do menu/about — bez zmian względem wcześniejszej wersji)
5. **neofetch w terminalu** — własny ASCII-art róży, jak opisano niżej

Jeśli chcesz zmienić, która tapeta jest domyślna, edytuj `last-image` w hooku `0100-set-wallpaper.hook.chroot` (obecnie wskazuje na `la-rosa-wallpaper.png`).

Jeśli zechcesz zmienić kolory (np. inny odcień bordo/różu), edytuj `style:` w `config/includes.chroot/etc/calamares/branding/la-rosa/branding.desc`, `active_text_color` w hooku `0300-branding.hook.chroot`, oraz `ascii_colors` w `etc/xdg/neofetch/config.conf` (kolejność: kwiat-ciemny, kwiat-jasny, kwiat-akcent, łodyga — numery odpowiadają standardowym kolorom ANSI: 1=czerwony, 2=zielony, 5=magenta, 13=jasny magenta itd.).

## Karty Wi-Fi Broadcom

W liście pakietów są: `b43-fwcutter`, `firmware-b43-installer`, `firmware-b43legacy-installer` i `firmware-brcm80211`. Pokrywa to większość kart Broadcom obsługiwanych przez sterowniki b43, b43legacy, brcmsmac i brcmfmac.

- `firmware-b43-installer` pobiera firmware z internetu podczas instalacji pakietu, czyli w trakcie `lb build`. Maszyna budująca musi mieć dostęp do sieci. Gotowe ISO działa już offline, a Calamares kopiuje firmware do zainstalowanego systemu.
- Część kart Broadcom (np. BCM4311/4312/4321/4322/43224/43225) działa lepiej na zamkniętym sterowniku `wl` (pakiet `broadcom-sta-dkms`). Nie dodałem go domyślnie, bo wymaga nagłówków jądra i koliduje z b43. Jeśli Twoja karta nie złapie sieci na b43, napisz model (`lspci | grep -i broadcom`), to dobierzemy właściwy sterownik.

## Ważne uwagi

- Chrome i WPS **nie są** w oficjalnych repo Debiana – dlatego wrzucamy je jako lokalne .deb do `packages.chroot/`.
- Jeśli Chrome lub WPS odmówi instalacji z powodu zależności – dociągnij brakujące pakiety i wrzuć też do `packages.chroot`.
- Instalator to **Calamares** w trybie live (`--debian-installer live`) — działa bezpośrednio z sesji live.
- Plymouth (ekran startowy przed zalogowaniem) ma zainstalowane pakiety bazowe (`plymouth`, `plymouth-themes`), ale nie ma jeszcze własnego motywu z logo La Rosa — obecnie użyty jest domyślny motyw Debiana. Jeśli chcesz, żeby splash startowy też pokazywał różę, daj znać, dorobię dedykowany motyw plymouth.
- Paleta kolorów: głęboki bordo/tło (#2a0710, #3d0d1a), czerwień-róż logo (#ff1654 → #c2117a gradient), akcent różu (#ff5c8a), złoto do subtelnych linii (#d9a94f).

## Struktura

```
la-rosa/
├── auto/config          ← główna konfiguracja live-build (XFCE + Calamares)
├── config/
│   ├── package-lists/   ← lista pakietów
│   ├── includes.chroot/ ← pliki wrzucane do systemu (tapeta, logo, os-release, hostname, branding Calamares)
│   └── hooks/           ← skrypty uruchamiane w chroot (tapeta, locale, branding, calamares)
├── .github/workflows/   ← budowanie ISO na GitHub Actions
└── README.md
```

Jak coś nie będzie działać przy budowaniu – wklej błąd, pomogę.
