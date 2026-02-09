#!/bin/bash
# i3/dmenu layered display manager
# Requires: xrandr, dmenu

# -----------------------
# List wired displays (X11 only)
list_wired() {
    xrandr --query | awk '/ connected/{print $1}'
}

# List available resolutions for a given display
# Output format: 1920x1080 @ 60.00
list_resolutions() {
    local output=$1
    xrandr --query \
    | sed -n "/^$output connected/,/^[^ ]/p" \
    | awk '/^[[:space:]]+[0-9]/{gsub(/[*+]/,"",$2); print $1 " @ " $2}'
}

# Ask for primary display if not set
get_primary() {
    primary=$(xrandr --query | awk '/ connected primary/{print $1}')

    if [[ -z $primary ]]; then
        primary=$(list_wired | dmenu -i -p "Select main/primary display:")
        [[ -z $primary ]] && exit
        xrandr --output "$primary" --primary
    fi

    echo "$primary"
}

# Choose position relative to primary
choose_position() {
    printf "left\nright\nabove\nbelow\nclone" | dmenu -i -p "Position relative to main:"
}

# Choose rotation for the display
choose_rotation() {
    printf "normal\nleft\nright\ninverted\nskip" | dmenu -i -p "Rotation:"
}

# Show done message via dmenu
show_done() {
    printf "%s" "$1" | dmenu -p "Done:"
}

# Configure a display
configure_display() {
    local output=$1

    # Check if active
    status=$(xrandr --query | awk -v o="$output" '$1==o{print $2}')
    [[ "$status" == "connected" ]] && current="on" || current="off"

    action=$(printf "Enable\nDisable\nCancel" | dmenu -i -p "Turn $output on/off? (Currently $current)")
    [[ -z $action || $action == "Cancel" ]] && return

    if [[ $action == "Disable" ]]; then
        xrandr --output "$output" --off
        show_done "$output disabled"
        return
    fi

    res=$(list_resolutions "$output" | dmenu -i -p "Resolution for $output:")
    [[ -z $res ]] && return

    resolution=$(awk '{print $1}' <<<"$res")
    refresh=$(awk '{print $3}' <<<"$res")

    primary=$(get_primary)

    rotation=$(choose_rotation)
    [[ -z $rotation || $rotation == "skip" ]] && rotation=""

    if [[ "$output" == "$primary" ]]; then
        xrandr --output "$output" --mode "$resolution" --rate "$refresh" ${rotation:+--rotate $rotation}
        show_done "Primary $output set to $resolution @${refresh}Hz${rotation:+, rotated $rotation}"
        return
    fi

    pos=$(choose_position)
    [[ -z $pos ]] && return

    case $pos in
        clone)
            xrandr --output "$output" --mode "$resolution" --rate "$refresh" ${rotation:+--rotate $rotation} --same-as "$primary"
            ;;
        left)
            xrandr --output "$output" --mode "$resolution" --rate "$refresh" ${rotation:+--rotate $rotation} --left-of "$primary"
            ;;
        right)
            xrandr --output "$output" --mode "$resolution" --rate "$refresh" ${rotation:+--rotate $rotation} --right-of "$primary"
            ;;
        above)
            xrandr --output "$output" --mode "$resolution" --rate "$refresh" ${rotation:+--rotate $rotation} --above "$primary"
            ;;
        below)
            xrandr --output "$output" --mode "$resolution" --rate "$refresh" ${rotation:+--rotate $rotation} --below "$primary"
            ;;
        *)
            return
            ;;
    esac

    show_done "$output set to $resolution @${refresh}Hz ($pos)${rotation:+, rotated $rotation}"
}

# -----------------------
main_menu() {
    options="$(list_wired)\nExit"
    choice=$(printf "%b" "$options" | dmenu -i -p "Select display:")
    [[ -z $choice || $choice == "Exit" ]] && exit
    configure_display "$choice"
}

main_menu
