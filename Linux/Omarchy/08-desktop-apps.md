# 08 — Desktop App Development (Omarchy)

**Profiles:** desktop · Same stacks and packages as `Linux/Arch/08-desktop-apps.md` (use `pkg` instead of
`pac`). Omarchy already has `qt6-base`, `qt6-declarative` and `qt6-wayland` (used by its shell), so those
show as present.

Hyprland tips for testing GUI apps: Wayland-native apps work out of the box. For X11-only toolkits
XWayland is on. Test scaling with `hyprctl monitors`.
