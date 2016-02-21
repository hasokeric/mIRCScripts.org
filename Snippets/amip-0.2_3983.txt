; amip-0.2
; pateketrueke@gmail.com
;
;
; Advanced mIRC Integration Plug-in (quick-addon)
; http://amip.tools-for.net/wiki/amip/download
; ------------------------------------------
;
;	-iN = Call the announce operation (current or 'N' preset, if defined)
;	-vNUD = Set the volume to 'N' value (or Up/Down respectively)
;	-sN = Set the playlist selected index (if sN is specified)
;	-sSNRN = By default toggle Shuffle/Repeat (set if 'N' is specified, respectively)
;	-s <...> = Set the output AmIP preset (without override the current)
;	-mW [/handler] <...> = Find or match (if 'W' specified <...> will be treated as regular expresion)
;	-cC = Clear the search results (if 'C' is specified clears the current cache-song-information)
;	-aN = Sets the autoplay mode (just after the find/match operation)
;	-jN = Jump-to-time ('N' must be in seconds)
;	-r = Reindex the current playlist
;	< = Plays previus entry (not as -flag)
;	> = Plays next entry (not as -flag)
;	-pN = Play (if 'N' is specified will be treated as playlist-index)
;	-o = Stops playback
;	-u = Pauses playback
;	-w = Rewind 5 sec.
;	-f = Forward 5 sec.
; ------------------------------------------
; You can mix some of these 'flags', even you know how works... (Ex. //amip -a1cCrmW $$?="Find-string-pattern")
;
;
;	The alias:
;		alias my_amip { tokenize 64 $1- | echo -s I'll be the AmIP current-song-info.... $0 > $1- }
;	Example:
;		//amip -s /my_amip $replace(&stat@&s@&name@&7@&fn@&fl@&typ@&min@&sec,&,%)
;
; NOTE: If you want to use other DDE name, you should change the variable value that contains the DDE name (by default is mPlug)
; ---------------------------------------------------------------------------------------------------------------------------
;
alias amip {
  var %amip.dde = mPlug


  if (!$isdde(%amip.dde)) {
    echo $color(info2) -s /amip: DDE server $qt(%amip.dde) doesn't exists
    return
  }


  if ($isflag($1,i)) {
    var %n = $nflag($1,i)
    if (%n isnum 1-5) dde %amip.dde announce preset %n
    else dde %amip.dde announce
  }

  ;
  ; Volume control
  ;
  if ($isflag($1,v)) {
    var %n = $nflag($1,v)

    ; if 'U' or 'D', sets Up or Down the volume (respectively)
    if ($isflag($1,U)) dde %amip.dde control vup
    elseif ($isflag($1,D)) dde %amip.dde control vdwn
    elseif (%n isnum 0-255) dde %amip.dde control vol %n
  }





  if ($isflag($1,s)) {
    ;
    ; Misc options (set)
    ;
    var %n = $nflag($1,s)
    if (%n isnum) {
      ;
      ; Set the playlist selected index
      ;
      dde %amip.dde setplpos %n
    }


    ;
    ; Toggle if 'N' isn't specified, otherwise set the value
    ;
    if ($isflag($1,S)) {
      ; Shuffle
      var %n = $nflag($1,S)
      if (%n isnum) dde %amip.dde setshuffle $st(%n)
      else dde %amip.dde control shuffle
    }
    if ($isflag($1,R)) {
      ; Repeat
      var %n = $nflag($1,R)
      if (%n isnum) dde %amip.dde setrepeat $st(%n)
      else dde %amip.dde control repeat
    }


    if ($2) {
      ; Current output preset
      dde %amip.dde set "" $$2-
    }
  }


  ;
  ; Playlist specific control
  ;



  if ($isflag($1,r)) {
    ;
    ; Re-index cached playlist (quiet)
    ;
    dde %amip.dde reindexq
  }

  if ($isflag($1,c)) {
    ;
    ; Clear search results and cache (if 'C' is specified)
    ;
    if ($isflag($1,C)) dde %amip.dde clearcache
    dde %amip.dde clear
  }

  if ($isflag($1,a)) {
    ;
    ; Sets the Autoplay option (for-playing-after the r?find API call)
    ;
    dde %amip.dde autoplay $st($nflag($1,a))
  }


  if ($isflag($1,m)) {
    ;
    ; Find
    ;
    var %handler = $iif($3,$2,/echo)
    var %string = $if2($3-,$2-)

    if (%string) {
      if ($isflag($1,W)) dde %amip.dde rfind %handler %string
      else dde %amip.dde find %handler %string
    }
  }



  if ($isflag($1,j)) {
    ;
    ; Jump-To-Time action
    ;
    var %n = $nflag($1,j)
    if (%n isnum) dde %amip.dde jumptotime %n
  }




  ;
  ; Back & Forward controls (the else avoid conflicts)
  ;
  if ($1 == <) dde %amip.dde control <
  elseif ($1 == >) dde %amip.dde control >
  else {

    ;
    ; Play control, or specific 'N' index (if any)
    ;
    if ($isflag($1,p)) {
      var %n = $nflag($1,p)
      if (%n isnum) dde %amip.dde play %n
      else dde %amip.dde control play
    }
    else {
      ;
      ; Plus controls (the else avoid conflicts)
      ;
      if ($isflag($1,o)) dde %amip.dde control stop
      elseif ($isflag($1,u)) dde %amip.dde control pause
      elseif ($isflag($1,w)) dde %amip.dde control rew
      elseif ($isflag($1,f)) dde %amip.dde control ff
    }
  }
}

;
; Stuff aliases
;

alias -l if2 return $iif($1 == $null,$2,$1)
alias -l nflag if (($isflag($1,$2)) && ($regex(nth,$1,$+(/,$2,(\d*)/)))) return $regml(nth,1)
alias -l isflag {
  if (!$regex(flg,$1,/^[-\+](?:\w+)?$/)) return
  if ($2) {
    var %i = 1, %c = $len($2), %m
    while (%i <= %c) {
      %m = $mid($2,%i,1)
      if (%m isalpha) {
        if (%m isincs $1) return $true
      }
      inc %i
    }
  }
}
alias -l st return $iif(!$1,off,$iif($1 != off,on,$1))

; amip-0.2