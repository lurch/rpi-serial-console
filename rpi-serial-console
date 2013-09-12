#!/bin/bash
# Script by AndrewS to make it easier to enable/disable the serial console
# (which allows the UART to be used by other programs)
# version 1.1 - better error handling
ACTION=$1
BAUDRATE=${2:-115200}
UART=ttyAMA0
CMDLINE=/boot/cmdline.txt
INITTAB=/etc/inittab
if [[ ! -f "$CMDLINE" ]]; then
  echo "$CMDLINE is missing" 1>&2
  exit 1
fi
if [[ ! -f "$INITTAB" ]]; then
  echo "$INITTAB is missing" 1>&2
  exit 1
fi
CMDLINE_STATUS=disabled
if [[ $(grep $UART "$CMDLINE") ]]; then
  CMDLINE_STATUS=enabled
fi
INITTAB_STATUS=disabled
if [[ $(grep -v ^# "$INITTAB" | grep $UART) ]]; then
  INITTAB_STATUS=enabled
fi
LIVE_STATUS=disabled
if [[ $(grep $UART /proc/cmdline) ]]; then
  LIVE_STATUS=enabled
fi
if [[ $CMDLINE_STATUS != $INITTAB_STATUS ]]; then
  echo "Inconsistent status! $CMDLINE is $CMDLINE_STATUS, but $INITTAB is $INITTAB_STATUS" 1>&2
  exit 1
fi
if [[ "$ACTION" != "enable" && "$ACTION" != "disable" && "$ACTION" != "status" ]]; then
  echo "Missing / incorrect command-line argument. Use enable, disable or status" 1>&2
  exit 1
fi

if [[ $ACTION == status ]]; then
  if [[ $CMDLINE_STATUS != $LIVE_STATUS ]]; then
    echo "Serial console on /dev/$UART will be $CMDLINE_STATUS after the next reboot"
  else
    echo "Serial console on /dev/$UART is $CMDLINE_STATUS"
  fi
else
  if [[ $EUID -ne 0 ]]; then
    echo "Serial console can only be ${ACTION}d by root user" 1>&2
    exit 1
  fi
  if [[ ${ACTION}d == $CMDLINE_STATUS ]]; then
    echo "Serial console is already ${ACTION}d"
  else
    if [[ ! -f "${CMDLINE}.bak" ]]; then
      cp "$CMDLINE" "${CMDLINE}.bak"
    fi
    if [[ ! -f "${INITTAB}.bak" ]]; then
      cp "$INITTAB" "${INITTAB}.bak"
    fi
    CMDLINE_CONTENTS=$(cat "$CMDLINE")
    TMPFILE="${INITTAB}.tmp"
    grep -v $UART "$INITTAB" > "$TMPFILE"
    if [[ $ACTION == enable ]]; then
      echo "T0:23:respawn:/sbin/getty -L $UART $BAUDRATE vt100" >> "$TMPFILE"
      mv "$TMPFILE" "$INITTAB"
      echo "$CMDLINE_CONTENTS console=$UART,$BAUDRATE kgdboc=$UART,$BAUDRATE" > "$CMDLINE"
    else
      echo "#serial getty on $UART removed by "$(basename $0) >> "$TMPFILE"
      mv "$TMPFILE" "$INITTAB"
      NEW_CMDLINE_CONTENTS=
      for p in $CMDLINE_CONTENTS; do
        if [[ ${p%%=*} == console || ${p%%=*} == kgdboc ]]; then
          if [[ $(echo ${p#*=} | grep $UART) ]]; then
            continue
          fi
        fi
        if [[ "$NEW_CMDLINE_CONTENTS" != "" ]]; then
          NEW_CMDLINE_CONTENTS="$NEW_CMDLINE_CONTENTS "
        fi
        NEW_CMDLINE_CONTENTS="$NEW_CMDLINE_CONTENTS$p"
      done
      echo "$NEW_CMDLINE_CONTENTS" > "$CMDLINE"
    fi
    REBOOT_NEEDED=
    if [[ ${ACTION}d != $LIVE_STATUS ]]; then
      REBOOT_NEEDED=", a reboot is required to make this take effect"
    fi
    echo "Serial console has been ${ACTION}d$REBOOT_NEEDED"
  fi
fi
