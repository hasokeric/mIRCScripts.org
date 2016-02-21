;	Regx - expressions viewer [pateketrueke@gmail.com]
;	usage: /regx (via command or status-popup)
;	"Allow you to view regular expressions.. "
menu status {
  -
  open Regx:regx
  -
}
dialog regx {
  title "Expressions viewer - [/regx]"
  size -1 -1 256 160
  option dbu notheme
  box "Match:", 7, 3 53 248 90
  button "&Execute", 3, 196 61 47 12
  check "Auto", 4, 191 81 21 8
  check "Eval", 5, 219 81 25 8
  check "Update on subject", 6, 191 91 54 8
  text "Subject:", 1, 2 39 21 8
  edit "", 10, 24 39 215 10, autohs multi return
  button "...", 11, 240 37 13 12
  list 12, 7 62 180 77, vsbar
  button "Close", 19, 2 146 39 12, cancel
  button "", 21, 0 0 0 0, ok
  button "Join && Copy", 9, 191 111 56 18
  button "Clear...", 22, 191 131 56 9
  edit "", 2, 2 3 250 30, multi return vsbar
}
on *:dialog:regx:*:*:{
  if ($devent == init) { did -f $dname 2 | did -c $dname 4 | noop $regex(regx,,/\C*/) }
  if ($devent == edit) {
    if (($did == 2) && ($did($dname,4).state == 1)) tregx
    if (($did == 10) && ($did($dname,6).state == 1)) tregx
  }
  if ($devent == dclick) {
    if ($did == 12) {
      var %l = $gettok($did($dname,$did).seltext,-1,160)
      if (%l) did -ra $dname 10 %l
    }
  }
  if ($devent == sclick) {
    if ($did == 3) tregx
    if ($did == 9) {
      var %cp = $didtok($dname,2,32)
      if ($did($dname,5).state != 1) %cp = $remove(%cp,$chr(32))
      clipboard %cp
    }
    if ($did == 11) {
      var %f = $sfile($mircdir,*), %l
      if ($isfile(%f)) {
        if ($fopen(regx)) .fclose regx
        .fopen regx $qt(%f) | regx -r
        did -ra $dname 10 $remove(%f,$mircdir)
        while (!$fopen(regx).eof) {
          %l = $fread(regx)
          if (%l) regx + %l
        }
        .fclose regx
      }
    }
    if ($did == 21) { tregx | haltdef }
    if ($did == 22) regx -r
  }
}
alias regx {
  if (!$dialog(regx)) dialog -m regx regx

  if ($1 == -r) did -r regx 12
  if ($2) {
    var %re, %i = 1, %c, %str = $2-, %t = $chr(160), %e = did -a regx 12
    var %s = %e %t, %r, %sb = %s %t, %expr = $didtok(regx,2,32)

    %expr = $iif($did(regx,5).state == 1,$eval(%expr,2),%expr)
    %expr = $remove(%expr,$chr(32))

    %s %str | %re = $regex(regx,%str,%expr) | %c = $regml(regx,0)
    %s %re ... $+($chr(40),%c,$chr(41)) $iif(%c <= 1,$+($chr(123) $regml(regx,1) $chr(125)),$chr(123))
    if (%c > 1) {
      while (%i <= %c) {
        %sb $chr(160) $+(\,%i) $regml(regx,%i)
        inc %i
      }
      %s $chr(125)
    }
    ;
    %s
  }
  return

  :error
  if ($dialog(regx)) {
    did -a regx 12 $chr(9) * Error in expression...
    beep | return
  }
  else echo -s $error
  reseterror
}
alias -l tregx .timertRegx -mo 1 260 regx -r $!did(regx,10)
;
