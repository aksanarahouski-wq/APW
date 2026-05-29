[00:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58) Richard Sacco: Um,  yeah,  I  read  through  most  of  it,  not  all  of  it.

[00:03](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3) Aksana Rahouski: All  right.  And  I  think,  again,  just  to  like,  ah,  the,  the  purpose  of  this,  right,  is  I  think,  um,  just  to  kind  of  overall  start  talking  about  what  they're  asking  for,  right?  And,  um,  like  obviously,  like  I  put  together  an  idea  of  what  new  configuration  management  system  could,  could  be,  right?  It  doesn't  look  anything--  because  I  was  like,  I  built  it  in  Lovable,  so  I  didn't  feed  it  any  kind  of  baseline  as  far  as  what  APW  look  like  today.  So  it's  purely,  uh,  is  there  for  just  a  kind  of  conceptual  reason,  just  to  like,  talk  through  kind  of  the  main  components  of  i-  th-  this  idea  and  whether  or  not  it's  able,  like  pursue  chasing.  And  I  would  say,  let's  maybe  kind  of  start  with  kind  of,  kind  of  setting  the  stage  with  like  what  configs  are  today  and  how  they're  managed,  um,  before  we  kind  of  pivot  to...  Because  what  I  realized  is  that,  too,  this,  th-  this  concept  is  completely  different  from  what,  how  configs  are  managed  today  and  built,  right?  Um,  a,  a  lot  of  it  is  also  not  fleshed  out.  Like  I  still  don't  know  how  like,  if  application  mechanism  will  be  the  same,  but  let's  just  kind  of  talk  about  now,  like  how  config  is  built  today  and  how  they  applied  to  devices,  like  how  devices  get  these  configs.  And  Richard,  I  know  you  kind  of--  I  have  like  a  current  framework  folder,  and  you  put  like  this  simplified  version,  which  basically,  I  think,  like  I,  I  read  through  yours  just  to  make  sure  I  didn't  miss  anything.  For  the  most  part,  it's,  it's...  yeah,  I,  I,  I-  I'm  aligned,  but  I  want  to  make  sure  we  as  a  team,  like,  are  on  the  same  page  as  far  as  like  h-  how  configs  are  managed  and  built  and  applied  today.  Uh,  so  maybe  I'll  ask  you,  like,  to  kind  of  give  like  a  bit,  like  a  quick-  unless  everybody's  like  up  to  speed  or  like  we  rec-  we  understand  like  how  configurations  are  stored  today  and  applied.  Mm-hmm.  A  high  level  run  or  flyover  wouldn't  be  bad.  Okay,  well,  let's,  let's  do  that.  Richard,  I'll  let  you  maybe  kind  of  lead  that  one.  I'll  share  my  screen,  or  you  feel  free  to,  and  if  it's,  if  it's  easier,  and  just  to  kind  of  basically  explain  to,  like,  how  these  configurations  are,  um...  Wait,  where  are  they?  They  are  under  admin.  Um,  browse  configuration,  and  maybe  we'll  have,  like,  an  add  configuration.  So  basically,  a,  at  a,  like,  configuration  level  as  an  entity,  like  we  can  kind  of  see  like  what  they  are,  how  they're  put  together.  Um,  okay.

[03:01](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=181) Richard Sacco: It's  fine.  Uh,  let  me  go  ahead  and  open  my  document.  So  basically,  a  configuration  is  something  that  you  edit  from  the  admin  back  end,  as  you  can  see  where  Oksana  is.  It's  basically  a  file  stored  on  S3.  It's  a  dot  dat  file,  generally.  Basically,  a,  a  bunch  of--  it  has  a  bunch  of  keys  and  values  that  are  separated  by  equal  signs.  If  you  wanted  to  dive  deeper  into  it,  but  as  you  can  see,  it's  that  it  has  the  config  group,  and  then  you  name  the  configuration,  and  then  the  anticipated  host  name,  the  carrier,  and  the  service  plan.  So  in  that,  in  that,  there's  a  sort  of  a  set  of  matching  as  it  is  right  now,  in  terms  of  like,  if  you  choose  a  config  group  that  relates  to  the  device  model,  if  you  choose  the  carrier,  that  relates  to  the  SIMs.  And  it  also  is  like,  it's  not  just  active  SIM,  but  active  and  present  sometimes,  uh,  factors  into  which  device  gets  which  config.  It's  all,  uh,  written  down  in  this,  like,  more  complicated  matrix.  But  so  it's  based  on  the  model,  it's  based  on  the  carrier,  and  it's  based  on  the  service  plan  that  it  can  apply  to.  And  basically,  the  scope  of  it  is  that  when  you  first  import  the-  just  to  describe  sort  of  the  customer's  flow,  is  when  the  customer  imports  devices,  right?  They  import  devices.  These  devices  don't  have  configurations.  They're  just  like,  they  have  their  m-  uh,  model  and  make  and  stuff,  but  they're  basically,  when  they're  first  imported,  they're  not  given  service  plans  generally  and  all  that  stuff.  All  that  stuff  is  generally  done  on  the  assign  page.  So  when  you  assign  a  device  to  a  company,  then  you  decide  its  service  plan,  and  then  that's  generally  where  a  configuration  is  assigned  to  a  device.  Um,  and  so  what  happens  after  a  configuration  is  assigned,  and,  you  know,  feel  f-  free  to  slow  me  down  if  I'm  just  breezing  through  this.  This  is  meant  for,  like,  very  quick.  Um,  but  basically,  when  a  configuration  is  assigned  to  a  device,  at  that  point,  uh,  a  job  is  sent  out  to  s-  to  change  the  device's  configuration,  and  we're  basically  calling  on  the  device  API,  the  general  router  API,  not  like  a  Verizon  API,  not  AT&T,  not  T-Mobile,  just  a  general,  uh,  device  API.  And  then,  and  we're  sending  that  configuration  up,  and  we  also  have  logs  on  any  messages  we  get  from  that.  It's  quite...  So  if  anything  happens  with  the  configuration,  we  should  be  able  to  kind  of  dive  into  it  a  little.  Um,  and  that  information  is  not  always  useful,  but-

[05:49](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=349) Stone Marballie: So  hold  on,  I  have  a  question.

[05:51](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=351) Richard Sacco: Yeah.

[05:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=352) Stone Marballie: So  the  user  creates,  or  the  admin  creates  the  config,  which  has  this  file,  which  has  all  these  parameters,  right?

[06:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=360) Richard Sacco: Mm-hmm.

[06:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=360) Stone Marballie: That  are  very  specific.  ...  Yes.  After  you  add  the  device  to  the  portal,  et  cetera,  and  you  select  the  service  plan,  model,  et  cetera-

[06:09](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=369) Richard Sacco: Mm-hmm.

[06:09](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=369) Stone Marballie: -you're  saying  that  the  portal  is  going  to  match  to  one  of  these  kind  of  figs-

[06:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=375) Richard Sacco: Yes

[06:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=375) Stone Marballie: ...  that's  closest,  and  then  use  that  file  that's  stored  in  the  S3  or  on  the  portal,  and  then  push  that  to  the  device,  right?

[06:23](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=383) Richard Sacco: Exactly.  Yeah.

[06:24](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=384) Stone Marballie: Okay.  All  right,  I'm  with  you.

[06:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=385) Aksana Rahouski: So,  like,  for  example,  like,  just  to  quickly  pause  you,  Richard,  there,  as  far  as,  like,  how  we  can  see  it  in  the  portal  today.  This  is  a  device,  right?  Device  uses  this  configuration.  We  can  l-  this  is  this  configuration,  so  if  we  can  look  at  it,  go  to,  like,  see  what  it  is,  we  can  edit  it.  We  could  see  that,  uh,  it's  this  group,  this,  um...  Uh,  okay,  so  this,  whatever,  like,  all  this  data  that  goes  in,  host  name,  configuration.  Here's  the  file,  and  I'll,  I'll  open  the  file  in  a  second.  Carrier  and  service  plans.  It  sounds  like  too  many  could  be  selected,  right,  Richard?  My  understanding,  like,  since  there-

[07:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=426) Richard Sacco: Yeah

[07:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=426) Aksana Rahouski: ...  are  quite  a  few.  Um,  so  it's  the  w-  and  they,  they  basically  like,  they,  they  build  the  file...  By  the  way,  this  is  what  the  file  looks  like.  Uh,  actually,  that's  not  the  file.  Let's  look  at  the  exact  file.  I  just  put  it  on  the  Three  above.  Um,  I  thought  I  did.  Where  did  I  put  it?  Hold  on.  Come  on,  did  I...  Project.  Oops,  sorry,  I  put  it  in  the  wrong  place.  Um,  configuration  one.  Goddammit!  I  have  to  go  find  him  now.  Okay,  here.  Here  what  the  file  looks  like.  So  a  file  itself  is  just,  uh,  six  hundred...  It's  between  six  and  seven  hundred  keys  where  they  set  the  values  for  them,  right?  They  said,  foo  equal  a,  b,  z,  blah,  blah,  blah,  key  value,  key  value,  key  value,  right?  So  they  specifically  craft  these  files,  a-  and  then  the  file  just  basically  gets  attached  to  the  entity  based  on  a  combo  of  like  group,  um,  carrier  service.  You  get  to  have  this  file.  And  then  device,  based  on  kind  of  matching  on  the  same,  based  on  what  carrier  you  are,  what  service  plan  you're  on,  it,  like,  automatically  matches  config.  Config  doesn't  get  assigned  to  a  device  by  kinda...  It,  it,  it  just  automatically  maps  to  this  config  based  on  the  device  data.

[08:50](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=530) Richard Sacco: Yeah,  and  then-

[08:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=532) Aksana Rahouski: So-

[08:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=532) Richard Sacco: Mm-hmm.

[08:53](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=533) Aksana Rahouski: Okay.

[08:53](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=533) Richard Sacco: And  because  it  happens  in  devices  after  save,  uh,  pretty  much  to  be  a  little  more  technical.  So  basically,  anytime  you  mess  with  a  service  plan  on  the  device-

[09:03](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=543) Aksana Rahouski: Yes

[09:03](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=543) Richard Sacco: ...  it'll,  it'll  say,  "Does  this  need  a  new  config?"

[09:05](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=545) Aksana Rahouski: Yeah.

[09:05](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=545) Richard Sacco: If  it  does,  it'll  come  back  config.

[09:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=547) Aksana Rahouski: Yeah.

[09:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=548) Richard Sacco: Yeah.

[09:09](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=549) Aksana Rahouski: Richard's  point  might  be  important  to,  like,  iterate.  If  I,  let's  say,  go  to  this  device,  I  changed  the  carrier,  which  by  the  way,  device  doesn't  actually  have  a  carrier.  It  just  have  SIMs,  right?  By  knowing  active  SIM,  that's  how  we  kind  of  derive  to  a  carrier.  But  let's  say,  for  whatever  reason,  this  device  would  become,  like,  it,  it  was  Verizon,  and  now  AT&T  or  T-Mobile,  right?  The,  the  minute  you  save,  what  the  system  does,  it  looks  for  a  new  match,  right?  It  says,  like,  "Okay,  based  on  all  these  parameters  that  I'm  gonna  send  in,  who's  your  carrier,  who's  your  service  plan,  and  whatever  else,  I  found  a  match  for  you,"  or  it  fails  if  it  doesn't  find  a  match.  So  basically,  the  goal  is  just  we  have  this,  like,  matching  algorithm  that  maps  device  based  on  what  configuration  device  has  to  a  config  files  based  on  what  configuration  that  config  file  has.  It's  not,  like,  a  strict  assignment.  It's  a,  like,  a  logical  assignment.

[10:14](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=614) Stone Marballie: Oh,  okay,  so  just  to  clarify,  though,  um,  we  are  matching  on  a  subset  of  parameters-

[10:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=622) Aksana Rahouski: Mm-hmm

[10:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=622) Stone Marballie: ...  that  are  on  the  device  model  in  the  database,  but  the  amount  of  configurations  that  could  change  is  more  than  that.  Because  you  just  mentioned  there's  over  six  hundred  different  parameters-

[10:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=635) Aksana Rahouski: Right

[10:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=635) Stone Marballie: ...  that  are  in  that  file,  right?  So-

[10:37](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=637) Aksana Rahouski: Nothing  to  stop  the  client  from  going  and  changing  this  file.  We  are  really  not  validating  against  this  file  at  all.

[10:44](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=644) Stone Marballie: Mm-hmm.

[10:45](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=645) Aksana Rahouski: Anything.

[10:45](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=645) Stone Marballie: Right,  but-

[10:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=646) Aksana Rahouski: Yeah

[10:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=646) Stone Marballie: ...  where  I,  what  I  was  alluding  to  is  that,  or  it's  more  of  a  question,  and  pardon  my  ignorance,  but-

[10:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=652) Aksana Rahouski: Mm-hmm

[10:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=652) Stone Marballie: ...  I'm  saying  that  you  could  have  two  variations  of  the  file  where  something  changes.  It  could  be  some  small  parameter  in  that  file  that  just  says-

[11:03](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=663) Aksana Rahouski: Mm-hmm

[11:03](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=663) Stone Marballie: ...  that  based  on  the  configurations  that  we  map,  could  really  map  to  both,  because  something  is  slightly  different  that  we  don't  track  on  the  portal  side.  Right?  In  theory.

[11:14](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=674) Aksana Rahouski: File  itself,  correct.

[11:16](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=676) Stone Marballie: You're  right.  So  that's,  that's  where  I'm,  you  know...  You  know,  I'm  just  thinking  out  loud.  Like,  how  would  we  handle  that  situation?  Or  maybe  you  guys  are  gonna  get  into  that,  but-

[11:27](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=687) Richard Sacco: Yeah.

[11:27](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=687) Aksana Rahouski: If  it,  if  it  finds  multiple  matches?

[11:29](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=689) Stone Marballie: Yes,  because,  you  know,  the,  you  know,  what  you're  using  to  trigger-

[11:34](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=694) Aksana Rahouski: Mm.  Yeah,  I  think-

[11:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=695) Stone Marballie: ...  the  device,  uh,  doesn't  match-

[11:37](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=697) Aksana Rahouski: Yeah

[11:37](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=697) Stone Marballie: ...  all  the,  you  know,  we're  not  tracking  all  the  capabilities  of  the  device  on  our  side.

[11:41](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=701) Aksana Rahouski: I  think  if  you're  asking,  like,  l-  literally,  let's  say,  if  this  device,  and  if  we're  just,  like,  simplified,  right,  we  take  carrier,  um,  carrier  model,  um,  service  plan,  right?  Three.  And  let's  say  we  would  have  two  different...  I  think  the  system  is  smart  enough  today,  it,  like,  it  has  to  be  like  a  dis-  a  combo  needs  to  have  unique.  It  will  not  allow  you  to  create,  like,  configurations  with  duplicates.

[12:12](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=732) Richard Sacco: ...  Yeah,  yeah.  Yeah,  that's  already  a  rule.

[12:14](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=734) Aksana Rahouski: Well,  that's  why  by,  by  kind  of  validating  that,  we  never  allow  a  scenario  where  a  device  could-  devices  config  could  lead  to  multiple  config  files.

[12:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=745) Richard Sacco: Mm-hmm.  And,  uh,  let  me  just  speak  to  another  thing.  There's  two  other  points  that  I  wanna  get  to,  and  then  I  think  I'm  gonna-  I'll  be  done.  So  the  other  point  is  that  another...  So  when  you  edit  a  device's  config  by  editing  properties  on  the  device,  a  configuration  job  gets  sent.  But  also,  the  most  common  way  for  a  configuration  job  to  get  sent  is  that  at  an  anticipated  host  name,  which  is  whatever  it  is,  VZ  only.  Basically,  when  a  check-in  comes  in  from  that  device,  we  have  to  make  sure,  based  on  certain  parts  of  the  check-in,  we  know  which  device  it's  for,  and  then  we  also  check,  does  the  host  name  match  the  anticipated  host  name  of  its  configuration?  Now,  if  it  does,  great,  it  has  the  right  config.  That's  what  our  system  concludes.  Now,  if  it  doesn't,  then  we  do  a  config  push  then  as  well.  So  there's  multiple,  uh,  times,  and,  uh,  I  think  it's  really  just  those  two:  if  a  device  gets  edited  or  a  check-in  comes  in  with  a  different  host  name.  Um,  and  then  the  other  point  I  wanted  to  get  to  is  this  sort  of,  like,  why  we  have  so  many  configs,  right?  Because  there  are  a  decent  amount  of  combinations,  first  of  all,  but  then  the  other  thing  is  there's  this  sort  of,  like,  company  hierarchy  to  everything.

[13:44](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=824) Aksana Rahouski: Mm-hmm.

[13:44](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=824) Richard Sacco: So,  so  for  each  company,  they  can  have  a  custom  config,  'cause  I  don't  know,  let's  say  in  a  more  real-world  scenario,  this  guy  has  a  jukebox,  and  it  needs  to  be  able  to  access  this  stuff  that  other  companies  don't  really  use  or  whatever,  so  they  need  their  own  special  config.  So  at  the  company  level,  there's  also,  like,  modifications  they  can  make  on  top  of  that  default  config  that's,  that's  there  as  well,  and  there's  also  this  system  where  a  distributor  can  say,  "Make  default  for  all  my  sub-customers."  Or  I  guess  this  would  only  be  handled  by  WAT  admins,  but  so  there  is,  like,  this  sort  of  config  hierarchy,  and  that's  really  why  we  have  so  many  in  the  system  now,  and  it's  unwieldy.  So  that's  really  the  problem  that  this  new  solution  is  trying  to  solve,  where  we're  sort  of  trying  to  take  these  key  pair  values  out  of  these  config  files,  these,  these  huge  config  files,  and  sort  of  organize  them  in  a  way  where  I  don't  have  to  manage  six  hundred  configs,  but  I  do  have  to  manage,  um,  these  key  pair  values  and  their,  and  their  certain  subsets.  Uh,  I  don't  know  if  that...  Does  that  make,  uh,  sense  to  everyone?  Did  you  get  lost  anywhere?

[15:04](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=904) Aksana Rahouski: Just  to  maybe  I'll  add  to  it  a  little,  the-

[15:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=907) Richard Sacco: Well,  but-

[15:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=907) Aksana Rahouski: -thing  on,  on  the  problem,  right?  So  because  today,  every  new  combination  of  a  configuration  needs  to  start  as  a  file,  right?

[15:17](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=917) Richard Sacco: Mm-hmm.

[15:18](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=918) Aksana Rahouski: And  let's  just  even  keep  it  as  simple  as  let's  assume  this  file  is  like  the  base  file,  right?  Like  a  global  file  that  every  con-  every  device  gets,  and  then  we  make  a  decision.  Let's,  let's  say  these  two  parameters,  let's-  I'm  just  picking  any,  like,  random,  right?  Needs  to  set  different  values  based  on  what,  let's  say,  carrier  that  is,  right?  So  for  AT&T,  it's  set  one,  for  T-Mobile,  it's  set  two,  for  Verizon,  set  three,  for  Dual,  set  four,  five,  right?  Now  it  means  I  have  to  create  five  copies  of  this  file,  and,  uh,  just  to  update  these  two  values,  everything  else  stays  the  same,  right?  Uh,  a-  and  again,  it's  simplifying  it,  right?  So  now  imagine  that  I'm  in  this  scenario  where  there  are  two  hundred  plus  files,  and  some  of  these  configs  are,  like,  totally  global.  They  are  set  in  every  single  file.  They  are  the  same.  Now,  let's  say  one  of  these  configs  change.  What  it  means,  that  I  have  to  open  two  hundred  files  and  change  two  hundred  files  and  make  sure  that  I  didn't  break  a  single  one  of  them.  It...  Scaling  this  becomes  impossible,  right?  And  now  imagine  that  on  top  of,  okay,  I  need  to  reset  it  for  every  carrier,  I  also  need  to  reset  it  for  every  service  plan,  and  then  there  are  these  ones  that  are  customer  ones.  For  a  specific  customer,  now  I  need  to  s-  reset  the  value  once  again.  If  I  have  hundreds  of  customers  and  fifty  of  need,  them  needs  custom,  I  need  to  create  fifty  more  files,  and  my  redundancy  grow  with  the  number  of  the  files.  Because  it's-  if  at  some  point,  I'm  a-  after  this,  like,  very  high  root  level  configuration,  that  is  always  key  value  hard-coded  in  two  hundred  files,  and  for  whatever  reason  that  changed,  I  have  to  update  all  my  files.  I  can't  just  update  one  value,  and  it,  like,  cascades  into  every  final  config  set,  right?  And  so  for  those  reasons,  we're...  Kinda  that's,  that's  our  main  problem  right  now,  is  that,  like,  each  config  starts  as  a  copy  of  yet  another  file.  Then  if  we  need  to  ever  change  it,  you  have  to,  like,  change  backwards  all  your  files.

[17:45](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1065) Richard Sacco: So  I  have  a  question.

[17:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1066) Aksana Rahouski: Mm-hmm.

[17:48](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1068) Richard Sacco: So  based  on  what  you  said,  right,  so  these  devi-  these  devices  take  a  file  with  all  these  parameters,  right?

[17:54](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1074) Aksana Rahouski: Yeah.

[17:55](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1075) Richard Sacco: Say,  say  five  hundred  and  eighty  fields  in  that  config  file  or-

[18:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1080) Aksana Rahouski: Yeah

[18:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1080) Richard Sacco: ...  global  never  gets  changed.

[18:02](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1082) Aksana Rahouski: Yes.

[18:02](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1082) Richard Sacco: Twenty,  there's  twenty,  twenty,  um,  key  value  pairs-

[18:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1086) Aksana Rahouski: Mm-hmm

[18:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1086) Richard Sacco: ...  that  are  customizable,  right?

[18:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1088) Aksana Rahouski: Yes.

[18:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1088) Richard Sacco: I  get  what  you're  saying,  that  where  you  can  go  and  change  every  variation,  right?  So  it  seems  to  me,  and  I  may  be  thinking  about  this  the  wrong  way,  that  where  we're  going  is  that  we're  going  to  dynamically-

[18:19](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1099) Stone Marballie: ...  generate  the  config  file  where  we  start  with  the  base  config-

[18:23](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1103) Aksana Rahouski: Yes

[18:23](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1103) Stone Marballie: -and  then  we're  gonna  track  all  these  other  configs  that  exist  in  the  database,  and  then  try  to  match  the  changes.

[18:29](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1109) Aksana Rahouski: Yeah.

[18:29](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1109) Stone Marballie: But  that  also  means  that  on  the  device  side,  right,  we  need  to  track  those  on  the  device  entity  so  that  we  can  search  them  against  the  config  and  then  build  the  full  config-

[18:40](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1120) Aksana Rahouski: Yes

[18:40](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1120) Stone Marballie: -because  we  start  with  a  base  config  plus  the  parameters  that  need,  and  then  dynamically  generate  a  file  and  then  push  that  file.  And  then  log  say,  "Okay,  this  is  the  config  that  was  generated,"  right?

[18:50](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1130) Aksana Rahouski: Exactly.  Yes.

[18:51](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1131) Stone Marballie: Right.

[18:51](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1131) Aksana Rahouski: Instead  of,  instead  of  configs  being,  uh,  like  a,  a  file  that  has  all  six  hundred  plus  values,  right?

[18:59](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1139) Stone Marballie: Mm-hmm.

[18:59](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1139) Aksana Rahouski: Um,  we  really  treat  it  as,  um,  at  the  end  of  the  day,  device  gets-

[19:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1146) Stone Marballie: Mm

[19:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1146) Aksana Rahouski: -a  set  of  configs,  right?  And  these  set  of  configs  are  dynamically  built  based  on  configs  coming  from  global,  coming  from  carrier,  coming  from  service  plan,  and  whatever  else,  right?  So  at  the  end  of  the  day,  yes,  the  goal  is  for  every  config-  for  every  device,  still  generate  this  res-  set  of  data.  It's  just  not  gonna  be  stored  in  the  file,  it's  gonna  be  stored...  Well,  with  the  solution  that  I'm  suggesting,  it's,  it's  we  start  storing  literally  key  values,  and  then  we're  just,  like,  building  this  entire  set  for  a  device  based  on,  like,  where  it's  coming  from.  So  we  walk  away  from  files,  there  is  no  file.  Like-

[19:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1192) Stone Marballie: Well,  well,  hold  on,  hold  on  there.

[19:53](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1193) Aksana Rahouski: Mm-hmm.

[19:54](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1194) Stone Marballie: Have  a  question.

[19:54](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1194) Aksana Rahouski: There's  no,  there  is  no  file  as  a  starting  point.

[19:57](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1197) Stone Marballie: Okay.  That,  that  part  I'm  agreeing  with  you.

[20:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1200) Aksana Rahouski: Yes.  I  don't  know-

[20:01](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1201) Stone Marballie: My  question  is  for  Richard  here.

[20:02](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1202) Aksana Rahouski: Mm.

[20:03](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1203) Stone Marballie: So  when  you  do  the  config  push  job,  right,  are  you  just  sending  a  JSON  to  it  or  you're  actually  sending  a  DAT,  DAT  file?  What  do  these  devices  respond  to?  The  content  of  a  actual  file  or  the  string  that's  inside  the  file?  See,  what  it  is.

[20:19](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1219) Richard Sacco: You  send  the  actual  file,  um-

[20:20](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1220) Stone Marballie: You  send  the  actual  file.  Okay,  so  we  have  to  still  be  in  the  business  of  making  a  file,  and  sending  it.

[20:26](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1226) Aksana Rahouski: I  have  a  question.

[20:27](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1227) Richard Sacco: Yeah.

[20:28](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1228) Aksana Rahouski: Uh,  uh,  is  it  an  actual  file?  Because  that's  where,  uh...  And,  and  again,  it  doesn't  matter.  At  the  end  of  the  day,  like,  the  input  for  a  device-

[20:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1238) Stone Marballie: Mm

[20:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1238) Aksana Rahouski: ...  the  shape  and  form  of  it  needs  to  be  a  file.  Still  nothing,  at  the  end,  it's  just  packaging,  right?  Whether  it's  JSON,  a  DAT,  whatever.

[20:47](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1247) Stone Marballie: Okay.

[20:47](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1247) Aksana Rahouski: Um,  I  do  have  a  question,  uh,  for  you,  Richard,  because  t-  the  way  it  works  today  to  some  of  these  configs,  right?  Again,  if  we  go  back  to,  uh,  um,  this  device  that  we're  looking  at,  right?

[21:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1260) Richard Sacco: Mm-hmm.

[21:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1260) Aksana Rahouski: Um,  when  we  apply  configuration,  we  start-  we  take  this  config  file,  right?

[21:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1266) Richard Sacco: Mm-hmm.

[21:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1266) Aksana Rahouski: And  that  is  where,  because  we're  allowed  to  build  base,  and  it-  maybe  this  was  like  a  custom  one  base  for  just  for  this  customer  or  whatever.  But  we  st-  uh,  device  today  also  store  some  of  the  configuration.  For  example,  um,  cellular,  backup,  and  Wi-Fi-

[21:21](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1281) Richard Sacco: Mm

[21:21](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1281) Aksana Rahouski: ...  these  are  actually  the  values,  the  config  values  that,  that  we  extract  these  and  inject  into  the  file  and  send  that  final  output,  right?  So  truly,  like,  our  data  that  cascade  as,  like,  default,  is  there  a  custom?  Yes,  if  custom  always  wins.  Does-

[21:39](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1299) Richard Sacco: Mm

[21:39](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1299) Aksana Rahouski: ...  does  device  itself  has  anything?  Yes,  Wi-Fi  settings.  Take  Wi-Fi  settings,  replace  from  the  file.  Does  the  result  still  a  file,  or  we  s-  or  are  we,  are  we  sending,  like,  a  JSON  or  something?

[21:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1312) Richard Sacco: No,  we  are  actually,  when  we're  sending  that  out,  we're  checking  if  that  device  has  those  special  JSON  stuff,  and  we're  editing  the  file  based  on  those-

[22:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1320) Aksana Rahouski: Okay

[22:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1320) Richard Sacco: ...  key  value  pairs  that  exist  in  the  JSON.

[22:03](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1323) Aksana Rahouski: Okay.

[22:03](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1323) Richard Sacco: So  it  is,  we're  sending  a  file,  we're  just  building  it-

[22:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1326) Aksana Rahouski: Okay

[22:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1326) Richard Sacco: ...  at  that,  at  that  time.  Uh-

[22:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1328) Aksana Rahouski: Okay.  So  I'll-  so  basically,  yeah,  if  a  device,  let's  say...  Again,  back  to  the  device.  If,  let's  say,  device  had  a  Wi-Fi  set  up,  and,  and  again,  I'm  just  like,  it's  not  correct,  but  ultimately  Wi-Fi,  let's  say,  build  this  set,  right?  So  we  will  just  re-replace  values  of  this  set  with  a  device  Wi-Fi  configs,  and  still  send  it  as  a  file  to  a  device.  So  that  DAT,  uh,  is  what's  going  to  an  actual  device.  Okay.

[22:42](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1362) Richard Sacco: Mm-hmm.

[22:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1366) Aksana Rahouski: Okay.  So  what  else  do  we  wanna  know?

[22:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1372) Stone Marballie: So  in  terms  of  the  dynamically  allocated  fields  that  we  are  going  to  have  the  freedom  to  manipulate,  how  many  fields  are  those?

[23:05](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1385) Aksana Rahouski: Um,  so  again,  a  lot  of  it,  this  is  where  it  might  be  a  little  bit  helpful  for  us.  Based  on,  um,  kind  of  analysis  I  ran,  just  to-  out  of  curiosity,  right?  Um,  like,  what's  the  volume  of  actual  keys  that  get  actually,  uh,  reset  in  these  files,  right?  Because  we  know  total  is  six  ninety-four.  A  lot  of  them,  m-  like,  nulls,  right?  Uh,  some  are  defaults  that  never  change.  Based  on  analysis  that  I  ran,  we're  roughly  at  like  h-  like,  half  of  them  have  nulls  set  up,  or,  like,  between,  like,  zeros  and  nulls.  Um,  so  really,  what  we're,  we're  seeing,  like,  again,  kind  of  broad  strokes,  right?  It's,  um,  three  hundred,  so,  like,  roughly  half  of  this  are  actually  the  ones  that  are  being  used.  So  out  of  six  ninety-four,  three  hundred  and  twenty  keys  are  actually  what's  being  set  in  these  files,  so  it's  half.  Is-

[24:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1455) Stone Marballie: Like,  how,  how  many  of  them  are  glo-  yeah,  that's  what  I'm  asking.  But  I'm  asking  you,  how  many  of  them  are  global  and  how  many  of  them  are  gonna  be  dynamic?

[24:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1462) Aksana Rahouski: Yes.  So,  um-...  So  with,  that's  a  good  question.  So  that's  what  a  client  gave  us-  this  is  what  client  gave  me,  us,  and  I  kind  of  took  their  Excel  file,  and  I,  again,  assuming  that  like  they,  they  think  about  it,  that  we  cascade  as  some  are  set  at  the  global  level,  some  are  at  the  model,  some  are  at  the  service,  customer,  device,  right?  So  it  cascades  like  this.  They  also  say,  like  based  on  my  research,  we're,  we're  also  seeing  that  some  of  these  parameters,  actually  value  could  be  set  in  global,  but  then  we  override  the  same  parameter  for,  let's  say,  model  I22  or  for  carrier  T-Mobile,  right?  So  like,  it,  it  like  it  becomes  some  of  these  keys  are  available  in  multiple  layers,  and  you  could  override  it.  And  with  that  assumption,  let's  say  you  have  a  key,  foo,  right?  It's  set  in  global,  but  also  we  did  override  it  for  T-Mobile.  Like  T-Mobile  always  win.  Like,  oh,  like  the  next  layer  overrides  the  p-  the  previous  layer.

[25:27](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1527) Stone Marballie: Okay.  So,  so-

[25:28](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1528) Aksana Rahouski: As  far  as  quantities,  uh,  that's  what  you're  asking,  right?

[25:31](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1531) Stone Marballie: Yeah.  Okay,  so  no,  you  answered  the  question,  um,  from  a  higher  level,  meaning  that-

[25:37](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1537) Aksana Rahouski: Yeah

[25:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1538) Stone Marballie: ...  it  doesn't  m-  we  can  consider  all  the  parameters  editable,  right,  as  a  starting  config,  and  then  we  can  overwrite  any  one  of  them  later  on  in  the  same  config  because  it's  the  last  processed  one,  sequentially,  re-  lives  on  the  device.  Okay,  so  that  makes  it  easier.

[25:55](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1555) Aksana Rahouski: Right.  And,  um,  as  I,  I  guess,  like  these  documents  kind  of,  they  don't  have  the  counts,  but  again,  based  on  what  they  gave  me,  and  again,  their  document  was  a  little  bit  of  a  happy  path,  right?

[26:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1568) Stone Marballie: Mm-hmm.

[26:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1568) Aksana Rahouski: They're  kind  of  like,  "Oh,  these  are  only  global,  and  these  are  only  carrier,  and  these  are  only  dev-  like  service  plan."  The  truth  is,  like  when  I  looked  at  this,  a  bulk  of  these  two  hundred  devices,  it,  it  actually  uh,  not  that  clean.  Like  overrides  exist.

[26:24](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1584) Stone Marballie: Mm-hmm.

[26:24](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1584) Aksana Rahouski: So  that's  why...  But,  but  like  kind  of  like  you  guys  can  scan  the  stages,  and  based  on  like,  uh,  what  they  gave  us,  and  I  think  again,  the,  the  truth  is  gonna  be  somewhere  like...  Okay,  they  gave  us  a  kind  of  good  starting  point,  but  the  truth  sits  in  these  files  that  we  have,  two  hundreds  of  them,  right?  Because  these  are,  these  are  actual  production  it-  configs  that  tell  us  what  the  final  set  for  global  model  and  service  and  such.  So  but  you  can  look,  like  we  don't  have  to  kind  of  zoom  in  on  a  lot  of  it.  It  will  have  to  be  finalized,  ah,  but  these  are  what  they  call  global,  so  there  are  a  bunch  that  they  gave  us.  I  also  kind  of  did  like  grouping,  because  I  do  think  since  we're  dealing  with  such  a  high  large  volume  of  them,  seven  hundred  of  them,  like  some  kind  of  grouping,  let's  say  these  are  like  firewall  ones,  these  are  WAN  interface,  these  are  LAN  interface,  Wi-Fi  configs  might  be  helpful.  So  you've  seen  my  prototype,  these  categories  and  configs.  Then  we  have  our  models,  so  these  are  the  ones  that  they've  identified  for  models,  service  plans,  customers,  and  such.  And  you  can,  again,  we  don't  have  to  kind  of  go  over  these  right  now,  but,  um,  the  data  is  there  available.  So  I  think  at  this  point,  let's  just  kind  of  jump  into,  uh,  and  use  my...  Unless  in  anything  else,  we  kind  of  need  to-  I  think  again,  like  looking  at,  um-  looking  at  the  prototype,  that's  where  it's  really  kind  of  helping  to  understand  the,  the  concept,  right?  Because  the  concept  is  very  different  from  what  it  is  today.  Um,  yeah.

[28:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1687) Stone Marballie: Before,  uh-

[28:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1688) Aksana Rahouski: Yeah

[28:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1688) Stone Marballie: ...  before  we  go  ahead,  um,  or  maybe  you  have  time  to  go  through  this,  so  how,  where  we  plan  on  tracking  this?  Are  we  creating,  um,  columns  on  the  devices  table,  and  then  we're  having  a  device  configurations  table  that  kind  of  maps?

[28:24](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1704) Richard Sacco: Yeah,  all  those  details  are  up  to  us.

[28:27](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1707) Aksana Rahouski: I-

[28:27](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1707) Richard Sacco: You  know,  this  is  more  issues  you're  showing,  like-

[28:30](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1710) Aksana Rahouski: Yeah.

[28:31](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1711) Stone Marballie: High  level.  Okay,  okay.

[28:32](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1712) Aksana Rahouski: I-

[28:32](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1712) Stone Marballie: Sorry,  my  brain  immediately  wants  to  go-

[28:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1715) Aksana Rahouski: Yeah

[28:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1715) Stone Marballie: ...  deep  down  into  the  dumps.  Sorry.

[28:37](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1717) Aksana Rahouski: Yeah,  and  maybe  like  it  makes  sense  to  kind  of  for  us  to  like  set  the  stage.  I,  I  don't  know  yet,  right?  I  don't  know  what  that  means  as  far  as  like,  um,  architecture  and  execution,  how  many  new  tables,  where,  what  data  we  store,  and  such  and  such.  I  just  like,  I,  m-  my  worry,  and  like  being  transparent  here,  when  I  kind  of  started  going  down  this  rabbit  hole,  right,  it's  just  like  became  so  complex,  and  it  sounds  very  expensive  to  me.  It,  which  it  is  fine.  You  know,  we  can  totally  go  like  pitch  it  to  them,  but  I  wanna  really  like  bring  more  heads  in  the  room  and  like  think  through  pros  and  cons,  and  maybe  I'm  overcom--  like,  maybe  it's  like  a  lot  more  complex  than  what  it  should  be.  Maybe  we  can  get  somewhere  on  middle  ground.  Um,  so  it's  really  just  the  concept.  I  don't  know  yet,  like,  where  everything  is  stored,  but,  um,  cloud-

[29:33](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1773) Stone Marballie: So-

[29:33](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1773) Aksana Rahouski: -give  like  a  lot  of  good  ideas,  and  you  can  read  through  that  entire  PRD,  which  is  now  a  book,  basically.  But,  um,  go  ahead.

[29:41](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1781) Stone Marballie: No,  no,  I'm,  I'm  thinking  from  the  perspective  of  like,  I  wonder  if  we  should  download  the  different  configurations  from  the  S3  bucket,  pass  the  files  into  cloud  and  say,  "Oh,  all  these  different  variations  and  configs  we  have-

[30:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1800) Aksana Rahouski: Yeah

[30:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1800) Stone Marballie: ...  which  are  the  ones  that  are  different?"  So  we  know  for  sure  against  what  base-

[30:04](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1804) Aksana Rahouski: Yeah

[30:04](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1804) Stone Marballie: ...  because,  you  know,  to  see,  okay,  so  these  are  the  subset-

[30:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1807) Aksana Rahouski: Yeah

[30:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1807) Stone Marballie: ...  that  we're  playing  with,  that  change,  right,  that  we're  trying  to  manipulate  to  get  the  variation.  Because  we're  trying  to  simplify  how  we  pick  them,  right?

[30:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1815) Aksana Rahouski: Well-

[30:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1815) Stone Marballie: The  second  thing  is,  what  happens  when  a  new  dynamically  one  gets  in?  Because  every  time  that  comes  out  with  a  new  firmware  release-

[30:26](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1826) Aksana Rahouski: Yeah

[30:26](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1826) Stone Marballie: ...  where  they,  you  know,  like  how  does  that  get  ingested  on  our  side?  You  know  what  I  mean?

[30:32](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1832) Aksana Rahouski: ...  No,  and  that's  like,  again,  that's  kind  of  one  way  to,  like,  let's  just  kinda--  I,  I  took  a  little  bit  different  approach.

[30:40](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1840) Noah Bratzel: Mm-hmm.

[30:40](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1840) Aksana Rahouski: I  kind  of  took  it  from  a  perspective  of  like,  well,  'ca-  'cause  like  I,  I...  Like  I  could  tell  you  kinda  how  I  started.  Like,  I  started  it,  and  I  was  like,  okay,  like,  let's  assume,  and  I'll  just  kinda  like  dive  right  in.  We  start  with  schema,  right?  Because  basically  what  we're  saying  is  just  we  need  to  define  a  schema  for  all  our,  like,  final  config  files,  which  is  six  hundred  and  ninety-four  keys  that  we  need  to  define  a  value  for,  right?  So  like-

[31:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1868) Noah Bratzel: Well,  no,  just  the  ones  that  are  gonna  change.  The  ones  that  are  global,  we  don't  need  to,  right?

[31:12](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1872) Aksana Rahouski: And-

[31:13](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1873) Noah Bratzel: We're  gonna  always  reuse  them.

[31:14](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1874) Aksana Rahouski: Correct.  You  can  simplify  that,  let's  say,  assuming,  like  back  to  our  files,  if  we  take  out  these  things  that  are  empty  today,  right?  And  not  worry  about  them,  and  we're  focused  on  only  on  three  hundred  and  twenty  that  are  in  fact  be  changed,  like  regularly,  right?  Maybe  we  build  it  just  for  this  three  hundred  and  twenty,  and  the  rest  of  them  kinda  sit  hidden.  We  just  always  kinda  append  to  it,  right,  at  the  end,  but  not  flash  it  out  through  the  UI.  Which  is  great,  but  then  back  to  your  kinda  second  question,  what  happens  if  these  two  that  are  always  empty  now  need  to  get  a  value?  We  need  to,  like,  change  the  solution  to  expose  this  tool  to  add  it  to  three  hundred  and  twenty.  Like,  wouldn't  we  rather  kinda  start  with,  "Okay,  here's  our  schema.  You  can  add  more  to  the  schema,  or  you  can  remove  to  the  schema,"  and  you  can  define,  like,  if,  let's  say,  for  a  field  that  is  never  used,  it  is  not  available  to  any,  any  layers.  Means  it's  only  exist  on  a  schema,  but  it  is  never  set.  It's  just  there  because  we  pa-  at  the  end  of  the  day,  what  we  build  is  just  this  DNN  access  equals  nothing.  No,  go  ahead.

[32:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1958) Noah Bratzel: No,  I'm  just  confused  about  that  whole  part  where  you  have  nothings  and  zeros  and  nulls.

[32:43](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1963) Aksana Rahouski: Uh-

[32:43](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1963) Noah Bratzel: Does  that  mean  the  same  as  not  having  it  there?  Is  that  how  it  works,  or  what's  the  deal  with  that?

[32:49](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=1969) Aksana Rahouski: So  that's  a  good  question,  and  I  don't  know,  but  per-  perhaps  it's  an  important  question  to  answer.  Why  these  configs  that  are  n-  nothing,  why  are  they  even  in  the  file?  Do,  does  the  device  need  to  receive  them?  Ok,  i-  if  we  take  this  file  and  strip  out  all  these,  like,  empty  things,  the  result  is  gonna  be  the  same.  And  I  think  it's  important  question  to  answer  up  front,  because  if  that  is  true,  if  stripping  out  these  things  out  of  my  schema,  meaning  and  not  sending  it  onto  the  device  with  no  values,  guarantee  the  same  r-  r-  result,  it's  like,  why  do  I  need  to  worry  about  them  in  the  first  place?  I  guess-

[33:36](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2016) Noah Bratzel: Yeah,  I,  I  think  that's  a  question  for  the  client.  I  don't  know  that  that's,  to  be  honest,  like  a  hundred  percent.  Maybe  we  should  just  focus  on  the  conceptual,  uh-

[33:47](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2027) Aksana Rahouski: Yeah.

[33:47](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2027) Noah Bratzel: Yeah.

[33:47](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2027) Aksana Rahouski: But  still,  I  think  it's  important  because-

[33:50](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2030) Noah Bratzel: Mm-hmm.

[33:50](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2030) Aksana Rahouski: A-  and  yes,  maybe  like...  I,  I  don't  know  if  like  something  on  the  device  side  must  receive  these  with  no  values  or  not  receiving  them  is  the  same,  right?  Like,  and  let's  just  kinda  table  that  one  as  that,  because  really  what  it  changes  for  us  is  just  the  size  of  the  schema,  right?

[34:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2048) Noah Bratzel: Okay.

[34:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2048) Aksana Rahouski: Because  then  we're  not  gonna  be  building  six  hundred  and  ninety-four,  but  we  will  be  building  just  three  hundred  of  the  ones  that  we  need  and  not  to  worry  about  the  rest  of  them.  But  I  wanna  kinda,  again,  like,  this  is  designed  with  a  concept  that  schema  could  change.  I  could  add  to  it,  and  I  could  remove.  I  could  add  a  new  key  to  my  schema,  and  w-  all  it  does  really,  at  the  end  of  the  day,  if  we  think  about  it  as  a  like  key  value  set,  it  just  adds  another  key  value  to  the  final  file.  Because  I,  I...  Again,  th-  this  allows  it  to  be  scalable,  right?  B-  but  again,  perhaps  this  is  something  where  we  draw  the  line  for  version  one,  right?  We  just  build  three  hundred  and  twenty,  and  we  freeze  it  at  that,  and  if  they  ever  need  to  add  another  key,  they  need  to  come  to  us.  Okay,  but  just  like  kinda  staying  on  the  schema.  Again,  so  really,  th-  this  is  designed  for  kinda  a  full  scalability,  right?  Like  I  start  with  six  ninety-four,  and  then  I,  I  have  an  ability  to  delete  them.  I  had  an  ability  to  add  new  ones.  Uh,  but  then  for  each  ski,  right,  uh,  I  specify  things  such  what  it  is,  uh,  what  category,  and  again,  like,  b-  based  on...  Because  since  it  might  be  easier  for  us  to  have,  is  it  Wi-Fi  settings?  Is  it  security?  Is  it  LAN?  Is  it  WAN?  Whatever,  right.  Um,  data  type  is  probably  something  is  important  because  if  we  wanna  have  any  kinda  like  validation  in  place,  but  again,  these  are  not  like  down,  down.  These  are  just  the  kinda  the  ideas  that  I  had  when  I  was  working  on  it.  Okay,  configuring  which  layer  it  can  be  defined,  right?  It--  this  one  is  available  in  global,  company,  and  device.  And  let's  say  this  one  is  available  in  all  six.  A-  and  what  that  means  really  is  that  this  key,  when  I  go  through  my  layers  builder,  right,  will  be  available  for  me  in  that  layer  to  set  a  value  on.  So  like  you  really,  you  kinda  like  knowing  that  we  have  this  multilayer  definition,  right?  It  allows  us  on  a  schema  level  to  define  which  level  it  is  available  on.

[36:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2195) Richard Sacco: ...  Just  I'm  curious  about  why  would  you  have  ones  that--  can  you  give  me  examples  of  ones  where  you  wouldn't  have  them  on  the  different  layers  and  why  that  would  be?

[36:48](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2208) Aksana Rahouski: Why  wouldn't  you  have  them  on  different  layers?

[36:49](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2209) Richard Sacco: Yeah,  why  wouldn't  they  just  be  on  all  layers?  I  just-  just  the  concept,  basically.

[36:53](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2213) Aksana Rahouski: Yeah,  and,  and  again,  that's,  that's  based  on  the  fact  that  client  even,  um...  This  is  the  doc  I  was  looking  at,  gave  us  this  document,  right?  Saying  that  these  ones,  we  only  ever  set  in  global,  right?  And  these  ones  are  v-  like,  it's  a  model  level.  Like,  we  set  it  for  every  model.  And,  um,  and  I,  again,  like  I  kind  of  what  I  did  when  they  gave  me  this,  and  I  didn't  look  at  it  like,  like  an,  an  entire  two  hundred  files  to  see  which  exists  on  which  level.  And  for  those  reasons,  I  think  if  you  build  it  into  the  solution,  like  rather  than  me  knowing  that  this  key  needs  to  be  in  level  one  and  two,  and  this  one  is  one,  two,  and  three,  build  it  into  the  solution  that  the  client  or  the  system  allows  you  to  figure  out  if,  if  it  needs  to  be  just  global,  you  define  it  to  be  just  global.  If  it  needs  to  be  global  and  let's  say  model,  meaning  I  could  s-send  it-  set  it  at  the  global,  and  I  should  be  able  to  override  it  for  i22.  It's  available,  right?  Like  I  could  do  that.  I  can  go  then  and  set  up.  So  like,  again,  trying  to  kind  of  scalability  in  mind,  knowing  that  it,  it  is  a  need,  in  fact,  for  the  same  key  to  be  available  in  multiple  layers,  so  we  just  make  this  like  layer  accessibility  feature  configurable  rather  than  hard  code  it.

[38:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2305) Richard Sacco: Yeah,  I,  I  just  to  quickly  try  to  summarize  this,  I  guess,  is  like  for  global,  you  can  decide  which  keys  are  global.  For  a  model,  you  can  decide  which  keys  will  be  put  under  the  model.  So  basically,  the  only  reason  you  would  have  keys  under  model  is  if  those  keys  are  basically  only  applied  to  i22s  or  only  applied  to  XYZ.  So  that's  really  the  idea  behind  that.

[38:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2332) Aksana Rahouski: Right.

[38:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2332) Richard Sacco: And  then  the,  the,  the  couple  issues  that  I'm  thinking  about  off  the  top  of  my  head  right  now  with  that  is  like,  let's  say  I  have  one  key  for  a  model  and  the  same  key  for  carrier,  and  if  it's  both  those,  now  which  one  wins?

[39:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2348) Aksana Rahouski: Yes.

[39:09](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2349) Richard Sacco: Uh,  yeah.

[39:10](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2350) Aksana Rahouski: Yeah,  and,  uh,  for...  And  that's  when  hierarchy  matters,  right?  If  we  go  like,  again,  I  just,  like,  like  quickly  kind  a,  of  finish  here.  Um,  configure,  like  all  the  flags,  again,  let's  just  think  about  like  on  a,  on  a  key  entity  level,  like  what  configs  key  gets,  right?  Is  like  what  category,  what  data  type,  what  layers  it's  available  in.  If  configuration  says  it  has  to  be  required,  let's  say,  right?  Uh,  maybe  that's  just  something  we  need  to  configure.  I  mean,  this  key  is  always  required,  so  must--  value  must  exist.  Maybe  some  default  value,  that  was  another  odd,  like,  idea  that  I  had,  because  some  of  them,  again,  if  you  look  here,  like  are  set  to  like  zeros,  right?  Like,  why  some  are  set  to  nothing  and  others  are  set  to  zero?  Is  zero  considered  a  default  value,  right?  I  don't  know.  Like  it--  and  again,  right  now,  kind  of  doesn't  matter,  but  these  are  kind  of  the  ideas  of  what  key  on  its  own,  like  what  configurations  key  could  carry,  right?  Um,  now  you  had  a  question  about  layer,  right?  So  like  now  we're  at,  um...  Okay,  it  cascades  like  this,  and  this  is  what  they've  been  trying  to  kind  of  explain  to  us,  that  what  you  just  said.  If,  let's  say,  we  have  a  global  config,  um,  key,  it's  set  to  zero,  and  then  on  a  model,  let's  say,  um,  whatever,  i22,  it's  overridden  to  one,  and  then  our  T-Mobile,  it's  o-  uh,  uh...  Okay,  so  I  do  think  what  you  just  said,  that's  actually  not  the,  um...  Okay,  this  is  where  it  gets,  like,  super  confusing.  Layers  can  override  just  global,  because  if  you  are  on  a  carrier,  let's  say,  level,  if  you  are  saying  that  you  need  to  override  i22,  you're  really  building  a  rule  here  that  you  are  saying,  "If,  if  my  device  is  i22  and  T-Mobile,  reset  this  key."  You're  not  resetting  global,  you're  resetting-  you're  building  this  like  two,  two-way  rule.  This  is  why  this  like...  There's-  this  concept  is  based  on  there  are  layers.  And  again,  let  me  kind  of  show  you  here  by  sam-  like  literally  looking  at  these  samples  that  I  put  together.  So  let's  say  default,  right?  All  the  fields,  keys  that  are  configured  for  default  were  set  values  here.  So  we  said,  okay,  value,  value,  value,  value.  Uh,  so  value  set.  Great.  Now  we  go  into  model,  right?  And  that's  where  for  every  model  that  they  wanna  set  the  value  for,  let's  say  i22,  they  could  choose  to  like,  assuming  that  LAN  zero  IP  is  configured  to  be  available  in  a  global  end  model,  you  could  choose  to  override  global  value  at  a  model  level.

[42:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2527) Richard Sacco: Oh,  wow!  Okay,  I  didn't  realize  you  had  that  built  in.

[42:10](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2530) Aksana Rahouski: Yeah.

[42:10](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2530) Richard Sacco: Oh,  wow.  Okay.

[42:11](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2531) Aksana Rahouski: Yeah.  So  it's  like...  So  and,  and  it's  the  same  true,  then  you  follow  through  carrier.  And  again,  as  you  can  see,  you  only  override  global  values.  You  don't  override  global  and  previous  layer  and  previous  layer.  This  is  what  the  rules  are  for.  So  really,  you  just  hear  here,  like-...  global  is  overridden  with  this  layer,  reset  new  value.  You  could-  you  get  to  see  kind  of  what  the  default  was,  if  it's  there,  and  you  get  to  override  it  at  this  level.  And  it  kind  of,  so  the  same  idea  then  follows  for,  like,  every  service  plan.  It  allows  them  to  truly  build,  like,  so  tier  one,  these  are  your  configs  that  are  custom  to  you.  And  again,  some  of  them,  keep  in  mind  that,  let's  say,  I,  I  think  I  tried  to  build  it,  but  I  don't  know  if  it's  like,  um...  Hold  on,  let  me  quickly--  Like,  like  this  one,  for  example,  right?  Let's  say  carrier  SIM  pin,  it's--  this  execution  is  not  the  best,  but  really  what  it  means  is  that  carrier  SIM  pin  is  only  configured  to  be  specified  at  a  carrier  level.  So  really,  I'm  not  overriding,  I'm  setting  it  because  it's  not  set  at  a  global.  This,  this  UI  is  not  perfect,  and  at  some  point,  I  had  to,  like,  stop  going  down  that  this  rabbit  hole,  right?  It's  just,  but  the  basically,  like,  as  soon  as  you  step  into  the  non-global  layer,  you  have  two  options.  You  have  either  the  option,  override  global  value  or  set  a  value  that  is  not  set  in  a  global,  that  has  to  be  set  just  on  the  carrier  level.  And  then-  so  like...  Go  ahead.

[43:50](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2630) Noah Bratzel: Well,  I  just  don't  understand  how  you  can  only  override  the  global,  'cause  what  happens-

[43:54](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2634) Aksana Rahouski: Yeah

[43:54](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2634) Noah Bratzel: -when  you  have  it  on  both  the  model  and  the  carrier  level?  You  can  have  a  key  in  with  those  two  levels.

[44:01](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2641) Aksana Rahouski: Correct.  And  this  is  where,  because  again,  for  this  particular  solution,  when  I  started  thinking  through  this,  right,  it  started  becoming  so  burly  because,  like,  for  example,  let's  say  I  would  have  I-22.  Like  here,  I  just  pick,  okay,  what  model  do  I  have,  right?  And  yes,  I-22,  these  are  your  values,  um,  four,  one,  zero,  these  are  your  values.  Now,  I'm  going  into  carrier.  Well,  now  I'm  saying...  And,  and  yes,  there  is  a  need  for  that  because,  like,  I--  this  is  where  I  kind  of  got  tripped,  and  I  was  like,  "Oh  my  gosh,  let  me  look  at  these  files  and  see,  do  I  ever  need  to  specify  value  for  I-22  and  T-Mobile?"  And  the  answer  is  yes,  and  it,  eh,  every  next  layer  for  a  service  plan,  you,  I  need  to  specify  for  I-22  and  T-Mobile  on  tier  one,  build  me  value,  key  value,  right?  But  in  this  interface,  I  was  like,  "Goddamnit,  how  am  I  gonna  now  build  it  to  where,  like,  for  a  service  plan,  uh,  tier  one,  how  do  I  build  a  rule  that  is  very  specific  for  a,  a  carrier  and  a  model,  right?"  And  I  pulled  it  out  into  this  rule  engine,  so  it's  basically  an-  another  layer  of,  like,  rules.  We  have  this  two-way  rules,  which  is  two-way  rule  means  it's  a,  any  combination  of  two.  It's  a  carrier  model  or  carrier  service  plan,  or,  um,  service  plan,  and  so  basically,  the,  the  three,  any  combination  of  the  three,  right?  Um,  and  then  there  are  three-way  rules,  right?  Like  model,  carrier,  service  plan.  I  will  say  that  the  good  news  is  that  three-way  rules,  there  is  never...  Like,  if  you  throw  in  a  customer,  right,  uh,  there-  we  don't  need  to  build  for,  like,  let's  say,  carrier,  service  plan,  and  customer.  It's  always  like  the-  and  I  don't  remember,  I  have  to  look  in  my  documents  like  that  I  did,  like,  analysis  and  research  are,  but,  like,  when  we  talk  about  three-way  rules,  it's  always  like  model,  carrier,  service  plan,  and  then  there  are  exceptions  for  customers.  But  really  what  you  do,  you  just  add  on  to  the  three-way  rule.  Like,  you  have  model,  carrier,  plan,  but  for  cord,  also  override  the  value.  So,  so  this  is  where  it  started  getting,  like,  really  complex,  but  again,  like,  based  on  kind  of  documents,  because  I  literally  took,  like,  config  files  for,  let's  say...  And  what  did  I  do  there?  I  did,  like,  run  this,  a  bunch  of  research  that  I  took,  like,  all  I-  I  like,  I  took  I-22  and  AT&T  and  Verizon,  right,  docs  and  different  customers,  and  I'm  like,  "Am  I  ever--  Do  I-  do  they  ever  reset  configs  for  combos,  right?"  And  the  answer  is  yes.  So  basically,  so  the,  as  soon  as  the  answer  is  yes,  that  tells  me  that  this  engine  and  or  con-  the,  this,  the  config  system  needs  to  be  allowing  to  build  these  rules,  right?  So  I  realize  that  this  is  a  lot  and  need  to  be  digested,  but  I  think  let's  just  quickly  look  at  the,  again,  back  to,  uh,  this  build  like  this,  which  basically  with,  um,  with  layers  in  mind  and  rules  in  between,  right?  We're  landing  with  this  eleven-levels  resolution  engine,  right?  That  starts  with,  um,  uh,  like  I,  I  mean,  it's  like  z-  kind  of  bottom  to  top  is  how  it's  set.  Like,  first  defaults  are  set  on  schema,  let's  say.  Global  parameters  are  set,  then  model  layer  overrides  it,  and  then  carrier  layer,  um...  Carrier  layer,  um,  ca-  ca-  could  bring  a  value.  And  then  we  have  our  two-way  rules,  and  then  service  plan  comes  in,  and  right  after,  the  three-way  rule  comes  in,  and  then  there's  company,  and  then  the  four-way  rule.  So  four-way  rules  would  be  something  such,  if  we  go  back  to  our  layers,  and  let's  go  to,  like,  a  company....  It's  gonna  explain  this  thing,  like,  explains.  So  we're  like  then  um  dynamically  building  configurations,  right?  For  a,  for  a  device.  We're  looking-  okay,  so  what  come...  What,  like,  what  are  your  like  inputs,  basically?  Carrier  model-  sorry,  um,  model,  model  carrier,  service  plan,  customer,  right?  It-  do  I  have  a  rule  specific  for  these  four?  No.  Moving  on.  Do  I  have  anything  specific  for  the  company?  Plug  them  in.  Do  I  have  anything  specific  to  a  carrier  service  plan,  um,  model?  Plug  them  in,  and  such  and  such.  So  it's  like  this  is  where  it's  like  it  becomes  like  complex,  right?  Somebody-  Aaron,  you  have  a  hand  up.  You're  on  mute.

[49:28](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=2968) Aaron Diefes: Sorry  about  that.  Um,  so  I,  I  think  I  understand  like  what  each  of  these  levels  means.  I  guess  my  question  is  like,  why,  why  do  the,  the  different  way  rules,  like  I,  I  even,  like,  for  example,  two-way  rules,  right?  Like,  if  you  had  a,  uh...  Let's  see.  Something  that  comes  after.  Like,  a  s-  a  service  plan  plus  customer,  right?  Like,  if  we're  defining  service  plan  as,  uh,  like,  that  higher  priority,  right,  like,  why  would  the,  that  two-way  rule  come  after  or  come  bef-  uh...  That  would  be  a,  uh,  like,  the  two-way  rule  would  be  a  lower  priority  than  like  the  general  just  for  service  plans,  if  that  makes  any  sense.

[50:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3008) Stone Marballie: Yeah.

[50:09](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3009) Aaron Diefes: Like  why-

[50:09](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3009) Stone Marballie: I  think  it's  just  because  to  be  more  specific  to  the  rule  that  you're  matching  in,  so  the  more  specific  your  rule  set,  you're  gonna  match  that  first.

[50:16](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3016) Aaron Diefes: Mm.

[50:16](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3016) Stone Marballie: And  then  if  not,  then  you  fall  back  to  a  higher  level  and  a  higher  level.

[50:19](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3019) Aksana Rahouski: Yeah.

[50:19](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3019) Stone Marballie: So  it's  kind  of  like  what  we  do  right  now  in  the  service  plan  when  we  pick,  um,  the  company  service  plan  tier  to  match,  right?  It's  kind  of  a  similar  situation.  You  know  what  I  mean?

[50:31](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3031) Aaron Diefes: I,  I  think  I  understand,  but  then  why  would  two-way  rules  be  a  l-  a  lower  priority  than,  like,  the  general  service  plan  layer?

[50:37](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3037) Stone Marballie: I  mean,  because  the  rule  after  it  also  includes  it,  right?  Because  it  could  have  the  two-way  rule,  but  then  the  three-way  rule  is  also  the  two-way  rule  plus  something  else.  Uh,  is  my  understanding,  right?  So  it's  like  a  more  specific.

[50:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3052) Aaron Diefes: Right.  That,  that  makes  sense  think  I  should  go  above  priority,  though,  then.

[50:55](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3055) Aksana Rahouski: And-

[50:56](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3056) Stone Marballie: True,  true.

[50:57](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3057) Aksana Rahouski: Yeah.  And,  and,  and  actually,  like,  that's  where,  right,  it's  not  final  yet.

[51:02](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3062) Stone Marballie: Mm.

[51:02](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3062) Aksana Rahouski: It's  kinda  like  perhaps  these  eleven  are  not  in  the  right  order,  and  we  need  to,  like,  truly  rethink  in  what  order.  Like,  when...  Because  yes,  basically,  we're  kinda  alternating  between,  like,  generic,  specific,  generic,  specific,  right?  And  then  these,  like,  rules,  they're  like  specific  to  a  very  specific  combinations,  right?  Every  time  you're  talking  about  for  this  specific  model  and  this  specific  carrier,  it's  a  two-way  rule.  Like,  does  it  come  in  before  or  after  just,  uh,  um,  just  the  carrier,  right?

[51:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3098) Aaron Diefes: Right.

[51:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3098) Aksana Rahouski: Uh,  and,  and  honestly,  like  I  said,  like  I've  kinda  ta-  like,  stopped  working  on  this  a  while  ago,  so  we'll  have  to  like  dive  in  and  specify.  And  for  now,  I  just  kinda  w-  wanna  show  you  the,  the,  the  concept  of,  right?  Like,  how  kinda  considering  the  layers,  layers  have  order,  right?  Order  matters  because  orders  is  how,  like,  we  resolve,  like,  who  wins,  right?  But  between  the  layers,  we  also  have  this  more  specific,  um,  combinations  of  exact  values,  right,  that  we  need  to  consider.  And  again,  maybe  there  is  a  good  way  to  solve  for  it  within  this  layer  framework,  right?  It's  just  like  I  said,  when  I  kinda  started,  like,  designing  it,  and  I  was  like:  Oh,  my  God,  it's  like,  how  do  I  now...  Because,  I  mean,  this  is  nice  and  cute  and  easy.  It's  one  for  all,  right?  This  already  gets  to  be  more  granular,  like  you  need  to  set  per,  right?  But  then,  like,  every  next  one  brings  per  carrier,  but  perhaps  per  model  that  was  before  the  carrier,  right?  And  this  is  where  this  rule  concept  kinda  define  itself  as  like  on  top  of  the  layers  where  values  just  said  specifically,  like,  if  you're  T-Mobile,  this  is  what  you  get  no  matter  what  customer  or  model  you  are,  right?  But  if  you  need  to  be  more  granular,  like,  it  matters  what  model  and,  and  carrier  you  are.  It  goes  under  the  rule  bucket,  and  it  allows  them  to  build  these  like,  uh,  rules  then.  So  they  have  to,  like,  specify,  right?  Like,  okay,  um,  like,  bas-  like  for  this  parameter,  we're  setting  which  one  is  it,  model  and  carrier.  Next,  which  model?  This  model.  Which  carrier?  This  carrier.  Boom!  What  is  the  value,  right?  So  it's  like  it  gives  you  this,  like,  based  on  what  type  of  rule  you  are,  it  gives  you  interface  to  build  the  proper  rule  and  allow  you  to  select  all  the  values  that...  And  obviously,  keep,  keep  in  mind  that  all  this,  like,  um,  the  list  of  carriers,  the  list  of  service  plan,  the  list  of  customers  will  be,  like,  fed  by  an  actual  list  of,  uh,  data  sets,  right?

[53:58](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3238) Aaron Diefes: Um,  yeah,  that  makes  sense.

[54:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3240) Stone Marballie: Mm-hmm.

[54:01](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3241) Aksana Rahouski: Um,  and  then,  so  okay,  conceptually,  are  we  all,  like,  tracking  as  far  as,  like,  are  we  all  on  the  same  page  as  far  as  how-  because  at  the  end  of  the  day,  right,  so  basically...  Yeah,  as  a  admin,  then  you  go,  and  you  basically  create  all  these  values,  right?  Now,  you  have-

[54:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3262) Aaron Diefes: Mm

[54:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3262) Aksana Rahouski: ...  full  control  over.  Your  schema  allows  you  to,  uh...  Oh,  another  thing,  I  think  on  a  schema  level,  you  actually,  uh,  um,  specify  if  this  is  like-...  um,  available  in  that  rule  engine.  Like  you  say,  this  key,  it  has  an  ability  to  be  set  in  a  four-way  rule  engine  or  three-way.  So  that  way,  like  we're  controlling  over  like  it-  because  if  it's  only  five  keys  that's  available  to  build  four-way  rules  for,  then  it's  only  five  that  we're  configuring  on  the  schema,  and  schema  kind  of  drives  w-  which  ones  are  available,  right?  So  again,  as  you  can  see,  kind  of  this  concept  is  based  on  the  fact  that  schema  drives  what's  the  list  of  fields  and  what  I  can  do  to  this  fields,  right?  Then,  then  you  have  like  a  concept  of  by  kind  of  schema  settings,  now  I  have  like  ah  layers,  and  I  can  set  these  keys  values  within  the  layers.  I  can  build  any  additional  rules,  right?  And  that's  like,  basically,  like  at  this  point,  we're  just  building  sets  of  values,  right?  Now,  the  question  is:  like,  how  do  we  grab  all  that?  And  like,  because  on  a  device  level,  right,  um...  and  this  is  where  like,  hold  on,  let  me  remind  myself  what  I  was  doing  here.  Like,  there  was  a  lot  of,  there  was  a  lot  going  on  when  I  was  working  on  this.  Um,  but  basically,  when  you  are  on  a  device  level,  right,  that's  where  I'm  gonna  be  the  big  shift,  and  I'm  not...  This  is  honestly  the,  the  one  that  I'm  kinda  the  most  fuzzy  on,  and  I'm  like:  "Well,  how  are  we  gonna  do  this?"  Because  also  remember,  device  could  carry  some  configs,  and  the  way  it  looks  today  is  like  this,  right?  They're  not  really  setting  up  key  values.  They're  configuring  a  form,  but  form  stores  key  value  set.  That's  from  a  customer  perspective,  right?  From  an  admin  perspective,  all  the  really  what,  what  we  need  to  worry  about,  like,  this  file  that  builds  configuration  is  not  gonna  be  like  a  physical  file  they  started  with.  It  needs  to  be  automatically  generated.  When  I'm  an  admin  and  I'm  looking  on  a  device  page,  right,  and  I  can  see  like,  this  is  the  final  set  of  configs  that  this  device  will  get.  I  do  want  tracking  as  far  as  where  it  came  from,  and  this  tells  me  that  this  came  from  global,  this  came  from  model,  this  came  from  a  rule,  da,  da,  da,  da,  da.  In  the  end  somewhere,  they  have  like,  we  need  some  kind  of  option  where  they  could  see,  view,  view  it,  and  what  viewer  gives  them  is  this,  right?  The  result  needs  to  be  this.  But,  but,  but  this  gives  them  tracking  to,  uh,  figure  out  where  was  this  configured,  because  that  way,  from  a  troubleshooting  and  testing  perspective,  they  can  quickly  go  like,  if  host  name  value  is  not  what  I  expected,  at,  at  least  it  tells  me  that  that  is  set  up.  Well,  let's  say  that  is  that  simple.  Let's  say  this  one,  like,  um,  this  is  not  what  I  expected.  Where  did  it  like,  it,  it  rendered  from  a  carrier  level,  and  I  can  at  least  then  know  like  where  it  was  set,  like  how  this,  basically,  the  re-  how,  how  resolution  algorithm  landed  on  a  value,  the  one.  Okay,  so  we're  not  gonna  solve  all  this  in  this  meeting.  The  goal  for  this  was  really  high.  Now,  I  wanted  to  guys  show  you  the  concept,  right?  How  do  we,  uh,  think  about  um  configs  as  a  completely  different  way  of  storing  them,  building  them,  keeping  them  dynamic,  right?  I  do  see,  like  again,  like  we  do  kinda  solve  the  main  pain  point  here,  is  like  if  they  ever  have  to  change  like  a  global  config  for  every  single  device  out  there,  all  they  do  is  just  they  go  to  their  global,  they  reset  this  value  once,  and  it  like,  it  gets  applied  to  every  single  device  because  every  single  device  uses  this  global  value.  They  do  not  need  to  edit  two  hundred  files.  Um,  still  a  lot  to  flash  out,  but  my  concern,  like  I  said,  is  just  like,  it's  looks  very  expensive  to  build,  and  a  lot  is  still  to  figure  out.  So  like,  give  me  your  thoughts,  like,  and  ideas.  Noah.

[59:04](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3544) Noah Bratzel: So  my  main,  my  main  thoughts  is  more  like  how  do  we  get  from  where  we  are  to  here,  and  especially-

[59:12](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3552) Aksana Rahouski: Yes

[59:12](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3552) Noah Bratzel: ...  with  you  talking  about  how  expensive  it  is.

[59:14](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3554) Aksana Rahouski: Yes,  that's-

[59:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3555) Noah Bratzel: And  so-

[59:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3555) Aksana Rahouski: Yes.  I  hear  you.

[59:16](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3556) Noah Bratzel: I,  I  was  going  to  think,  w-  like  when  I  was  thinking  through  this,  is  that,  that  it  would  be  nice  if  we  could  start  smaller  with  like  one  of  the  smallest  parts-

[59:27](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3567) Aksana Rahouski: Mm-hmm

[59:27](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3567) Noah Bratzel: ...  of  this.

[59:28](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3568) Aksana Rahouski: Mm-hmm.

[59:28](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3568) Noah Bratzel: And  I  was  thinking  if  it  would  it  be  possible  to  just  do  the  part  where  we're-

[59:34](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3574) Aksana Rahouski: Mm-hmm

[59:34](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3574) Noah Bratzel: ...  we're  doing  the  dynamically  created,  um,  configs-

[59:42](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3582) Aksana Rahouski: Yep

[59:42](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3582) Noah Bratzel: ...  but  still  have  it  work  the  way  that  it's  working  now.

[59:45](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3585) Aksana Rahouski: Yes.

[59:45](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3585) Noah Bratzel: When  we  set  up  the  whole  schema,  we  set  it  up  that  way.  So  we-  phase  one  would  be,  hey,  we're  not  really  changing  the  logic  that  it's  doing,  but  we're  not  doing  these  files  anymore.  Now  we  have  the  schema,  we  have  these  rules.  Then  by  having  that,  you  could  kind  of  actually  do  it  almost  no  risk  of  when  you  swap...  You  know,  you  put  that  in  place,  and  then  you  can  actually-

[01:00:04](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3604) Aksana Rahouski: Yeah

[01:00:04](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3604) Noah Bratzel: ...  compare  and  say,  "Okay,  those  are  actually  the  same."

[01:00:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3607) Aksana Rahouski: Yeah.

[01:00:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3608) Noah Bratzel: And  then  actually  switch  over,  get  rid  of  the  files,  boom!  We  have  that  in  place.  Then  you  can  also  add  it  to  auditing  and  stuff  like  that  at  the  same  time.  But  when  things  change,  you  know,  do,  do  all  that  layer  first.  It  seems  like  a  much  cheaper,  quicker  thing  to  do  and  safer.  And  then-

[01:00:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3625) Aksana Rahouski: Yeah

[01:00:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3625) Noah Bratzel: ...  phase  two  could  be,  okay,  what  is  the  most  important  layer  to  put  on  top  of  this?  And  do  one  layer  at  a  time  type  of  thing.  That's  kind  of  what  I  was  thinking  when  I  saw  this.  I,  not  that  I  don't  like  the  whole  complex  thing,  but  it's  a  lot.

[01:00:37](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3637) Aksana Rahouski: It  is  a  lot.

[01:00:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3638) Noah Bratzel: It's-

[01:00:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3638) Aksana Rahouski: Yeah,  and  I  see  that,  too.  I  will  say,  and  I  definitely  am-...  and  that's  why  I  need  you  guys,  because  I  often  struggle  with  like  simple  to  complex.  I'm  just  like,  solve  for  everything,  right?  I  will  say  the  good  thing  is,  like,  I  think  what's  important  to  highlight  we  are,  uh,  the,  like,  and  it's  i-  in  fact  requirement.  This  is  gonna  be  something  brand  new.  The  expectations  is  that  w-  at  some  point,  the  portal  will  have  two  different  config  systems:  the  old  one  and  the  new  one.  And  we  need  to  have  an  ability  to,  uh,  migrate  only  one  device  or  one  customer  in  this  new  concept,  and  should  be  able  to  push  them  back.  So  think  about  like,  like,  uh,  so  the-  they're  definitely  realizing  with  the  complexity  of  this,  it  cannot  be  all  or  nothing,  right?  So  really,  what...  Which  I,  I  think  it's,  it's  a  perfectly  like  a  perf-  uh,  this  is  perfect  because  it  actually  allows  us  to  start  small.  That  if  we're  only  saying  that,  okay,  that's  like,  let's  like  half-bake  it,  right?  What  does  half-baked  version  look  like?  And  with  the  goal  of,  like,  at  the  end  of  the  day,  like,  we  should  be  able  to  come  to  the  device  and  somewhere  here  specify,  uh,  use  the  old  configuration  or  the  new  configuration,  right?  And  assuming  that,  let's  say,  for  this  device,  we  build  an  entire  config,  in  the  switch  all  it  does  is  just  this,  uh,  file  is  built  by  a  new  system  instead  of  the  old  system,  right?  And  it  allows  us  to  kinda,  by  having  two  systems  in  place  in  parallel,  it  allows  us  to  kinda  not  to  have  this,  like,  hard  cut  over.  Like,  when  we  launch  this  thing,  we  better  hope  and  pray  that  every  single  device  is  gonna  build  the  right  configs.  It  allows  us  to  move  as  slow  or  as  fast  if  we-  as  we  can,  because  we  will  be  migrating  just  first,  probably  one  customer  or,  like,  one  device,  maybe  one  customer,  then  maybe  half  of  them,  I  don't  know.  But  it's  like,  truly  think  of  it  as  like  th-  this  is  gonna  be  like  a  brand-new  addition  nobody's  using  now.  So,  like,  our  current  system  shouldn't  rely  on  this  thing  until  it's  somewhat  ready  for  our  first  c-  uh,  device.  Which  I  think  this  is,  this  is  great  in  my  opinion,  because  in  this  case,  right,  it's  like  reversible.  Because  if  we  then,  like,  build  it,  let's  say,  and  we're  ready  to  put  our  first  device  on  it  and  test,  right,  and  something  didn't  work,  we  could  reverse  it  back  to  the  old  mechanism,  and  at  the  end  of  the  day,  they  have  their  physical  file,  it  gets  resolved  the  same  way.  We  know  it  works,  right?  And  it  allows  us  to  kinda  go  back  and  forth  until  we're  at  a  point  where  we  feel  like  this  new  solution  is  solid,  and  we  can  migrate  everybody  off  it,  and  some  said  the  old  configs.  I  guess  I  would  need  to  understand  a  little  bit  more  when  you  said  we  start  with  just,  um,  schema,  because  if  we  start  just  with  schema,  and  schema  is  where  you  set  values,  that  tells  me  that  every  device  needs  to  have  a  dedicated  schema,  which  is-  or  set  of  values  for  that  schema,  which  is  no  different  from  files,  like  how  we  exist  today,  right?

[01:04:14](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3854) Noah Bratzel: That  would,  that  would  be  the  idea,  is  that  you  would  keep  it  working  the  way  it  is.  You're  basically  taking  the  files,  and  you're  building  templates  off  of  the  files  that  use  the  schema,  but  you'd  be  moving  it  over  to  use  the  new  system  before  the  new  system  is  in  place,  basically.  You  would  keep-

[01:04:28](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3868) Aksana Rahouski: But...  Yeah,  but  if  you...  I  th-  I  think  with  your-  I  mean,  you're  the  old  pr-  like,  it's,  it's  a  different  concept,  though.  So  you  were  saying  we  just,  let's  just  take  this  file,  right?  And  let's  just  basically  put  it  in  a  form,  right,  and  allow  them  to  build  it.  Conceptually,  though,  it  doesn't  change  that  for  every,  like,  you  have  two  hundred  files  or  schema.  Like,  i-  instead  of  living  in  the  file,  it's  gonna  live  in  the  database,  and  if  you'd  have  to  change  global  value,  you  still  have  to  ha-  go  to  two  hundred  instances  of  that  thing-

[01:05:05](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3905) Noah Bratzel: Yeah

[01:05:05](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3905) Aksana Rahouski: -and  change  it  two  hundred  times.

[01:05:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3907) Noah Bratzel: Because  that's  what  I'm  saying  is  you're  not  so  trying  to  solve  that  problem  on  the  first  step.

[01:05:12](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3912) Aksana Rahouski: Right.

[01:05:12](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3912) Noah Bratzel: You're  trying  to  get  over  to  the  system  on  the  first  step,  and  then  you'd  have  the  auditing,  and  you'd  have  your  schema  system,  and  then  you  would-

[01:05:19](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3919) Aksana Rahouski: Yeah.

[01:05:19](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3919) Noah Bratzel: This  is  the  way...  What  I'm  describing  is  how  you  do  this  slowly  and  safely,  but  that,  that,  that  isn't  taking  into  account  what  you've  said  about  having-

[01:05:28](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3928) Aksana Rahouski: Mm-hmm

[01:05:28](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3928) Noah Bratzel: -two  systems  running.  I  would  think  of  it  as,  uh,  as-

[01:05:30](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3930) Aksana Rahouski: Yes

[01:05:31](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3931) Noah Bratzel: ...  get  on  a  new  system  that  works  exactly  the  same,  and  then,  then  you  add  the,  the  layers  that  simplify  that  system  that's  already,  you  already  gonna  know  is  working,  and  then  you-

[01:05:41](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3941) Aksana Rahouski: Yeah

[01:05:41](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3941) Noah Bratzel: -simplify  the  system,  but  it  to,  to  work  differently,  gradually,  one  piece  at  a  time.

[01:05:47](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3947) Richard Sacco: For  example,  Noah,  you  were  saying  maybe  we'll  build  the  UI-  -and  then  just,  like,  and  then  they  could  verify  on  the  UI  whether  the  config  is  gonna  be  the  same  as  it  is  currently,  and  then  it's  like,  so  you  can  switch  over  that  device  or  something.  Is  that  what  you're,  kinda  like  the  path  you're  talking  about?

[01:06:05](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3965) Aksana Rahouski: I  think-

[01:06:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3967) Noah Bratzel: Uh-

[01:06:07](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3967) Aksana Rahouski: Go  ahead,  Don.

[01:06:10](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3970) Noah Bratzel: Uh,  I  don't  know.  I  don't  know,  but  I  haven't  really  thought  through  the,  the  UI  part  at  all.

[01:06:14](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=3974) Aksana Rahouski: I,  I  think,  I  think  what  Noah  is  saying,  it's  almost  like  working  backwards,  right?  Assume  that  we  store  this,  like,  key  values  in  the  database.  Can  we  generate  a  file,  push  that  file?  Uh,  uh,  validate  that  that  file  is  identical  to  the  file  that's  been  created  today,  push  it  into  the  device.  So  it's  like  a  final  mile  kind  of  approach,  right?  My  issue  with  that  is,  like,  to  me,  it's  not  a  production-ready,  because  the  complexity  of  this  i-  is  not  the  schema.  It's  in  fact,  this  layered  and  rule  engine  and  the  hierarchy.  That's  the  most  complex  thing  here,  not  the  schema  itself.  Like,  if  they,  if  they  just  needed  a  simple,  like,  uh-...  give  me  like  some  kind  of  presentation  in  the  UI  for  this  file  that  I  never  get  to  see.  That's  not  what  we're  solving  for  because  I  still-

[01:07:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4026) Noah Bratzel: Okay.

[01:07:06](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4026) Aksana Rahouski: -  fundamentally,  we  still  have  the  same  problem  then,  that  if  something  global  changes,  it's  two  hundred  times  that  I  have  to  do  the  change,  not  once.

[01:07:17](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4037) Noah Bratzel: Yeah,  but  see,  okay,  but  I'm  not  trying  to  solve  that  problem  yet.

[01:07:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4042) Aksana Rahouski: Mm-hmm.

[01:07:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4042) Noah Bratzel: I'm  trying  to  get  to  a  place  where  we  can  safely  solve  the  problem.

[01:07:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4045) Aksana Rahouski: And  that,  and  that's  fine  if  that  we  take  it.  I  would  just  say  the  only  thing  that  that  would  not  be  like  a  production-ready  release,  because  we  are  adding  no  value  to  this  by  doing  that.

[01:07:36](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4056) Noah Bratzel: You're-

[01:07:36](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4056) Aksana Rahouski: Just  that,  just  that.

[01:07:37](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4057) Noah Bratzel: You're  absolutely  adding  value.  You're  adding  the  auditable-ness  of  it.  You're  cha-  changing  the  structure  of  it.

[01:07:43](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4063) Aksana Rahouski: Right.

[01:07:43](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4063) Noah Bratzel: You're,  you  could  get  rid  of  the  files.  You're  completely  changing-

[01:07:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4066) Aksana Rahouski: We  have  no  problem  editing  files-

[01:07:48](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4068) Noah Bratzel: Just-

[01:07:48](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4068) Aksana Rahouski: ...  let  me  make  it  clear.  They,  they  don't  have  a  problem  with  editing  files.

[01:07:51](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4071) Noah Bratzel: It's  a  huge  value  is  that,  that  you're  a  step  towards  the  system.  This  is...  And  the  point  is  not  that  I'm  not  saying  that  we  do  exactly  what  you're  saying.

[01:08:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4080) Aksana Rahouski: Yeah.

[01:08:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4080) Noah Bratzel: I'm  saying  how  you  get  there.  What  you're  describing  is  something  that's  gonna  take  months.

[01:08:04](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4084) Aksana Rahouski: Yes.

[01:08:04](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4084) Noah Bratzel: What  I'm  describing  is  something  w-  that  I  could  do  in,  in  relatively  short  terms-

[01:08:10](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4090) Aksana Rahouski: I  know

[01:08:10](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4090) Noah Bratzel: ...  and  have  something  rolled  live,  and  then  we  could  do  another  s-  step  in  relatively  short  terms,  almost  you  might  say  Agilely.

[01:08:17](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4097) Aksana Rahouski: And,  and,  yeah,  listen,  I,  I  hear  you.  The  only  thing  I'm  pushing  back  on  is  that  I  would  not  wanna  put  up  an  actual  device  on  the  new  system  at  that  point  yet.  I  would  maybe  temporarily  put  it  on  it  to  validate  that  it  works,  but  I  would  move  it  right  off  it  as  soon  as  I  was  tap  into  working  on  layers  and  rules.  Because  the  layers  and  rules  is  the  main  complexity  of  this  thing,  not  the  schema  itself,  and  by  having  devices  already  using  it  and  bu-  building  the  heartbeat  of  this  thing,  it's  extremely  risky.  However,  I  do  agree  with  you  that  actually  that's  probably  how  I  would  start  it  also.  But  just  to  kind  of  validate,  like,  does  this  work?  Yeah,  layer  two.  Let's  see  what  layer  two  is.  But  I  would  like  the  minute  we  put  devices  on  this  s-  thing,  and  now  we  have  to  worry  about  not  breaking  these  devices  while  we're  actively  building  the  most  important  thing  of  the  system.  That's  my  only  concern.

[01:09:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4165) Noah Bratzel: Yeah,  I  mean,  the...  No  matter  what,  we  have  to  be  very  careful  not  to  break  things.  But  yeah,  it  would  be-

[01:09:30](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4170) Aksana Rahouski: Yeah.

[01:09:30](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4170) Noah Bratzel: What,  what's  nice  about  is  you  could  s-  you  would  know  more  as  you  go.

[01:09:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4175) Aksana Rahouski: Yeah.

[01:09:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4175) Noah Bratzel: The  thing,  the  thing  that  I  don't  love  about  something  this  big  and  complex,  and,  like,  even  trying  to  understand  it  all,  is  it's,  it's  you're  building  this  huge  book,  or  document.

[01:09:45](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4185) Aksana Rahouski: It  is,  yeah.

[01:09:45](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4185) Noah Bratzel: And,  like,  when,  w-  what  I'm  proposing  is  if  after  step  one,  you  can  review  and  look  at  it  again  and  say:  "Okay,  this-"

[01:09:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4192) Aksana Rahouski: Yeah.

[01:09:52](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4192) Noah Bratzel: "-step  two  now  that  we  would  take,  is  it  still  making  sense?  Is  it  easier  to  understand?"  It  would  be  way  easier  to  understand.

[01:09:58](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4198) Aksana Rahouski: Yeah,  i-  i-  and  I'm  not  denying  that.  I  agree  with  you.  I'm  just  saying  that,  and,  and  I  think  you're  saying  that,  too,  that,  like,  after  we  do,  like,  let's  say,  let's  say  we,  like,  fin-  we  have,  like,  our  ten-milestone,  um,  plan,  right?  We  d-  we  do  ten  releases  to  get  to  the  final  product.  What  I'm  just  saying  from  what  you  described,  the  mile  number  one  looks  like,  I  would  not  recommend  us  yet  treat  it  as  we  can  put  a  p-  a  device  on  it,  and  we  can  keep  it  on  it  until  we  cross  the  final  mile.

[01:10:32](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4232) Noah Bratzel: Yeah.

[01:10:32](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4232) Aksana Rahouski: I  would  say  let's  put  it  on  it,  sniff  test  validated,  move  it  back,  keep  building  mile  two.  The,  the  beauty  of  it  is  because  we  building  some,  like,  a  brand-new  addition  that  nobody  uses  it,  we  can  kind  of  test  around,  like,  let's  move  somebody  there,  see  if  they're  comfortable,  move  them  back  out  until  it's  under  construction,  right?  Um,  Aaron,  did  you  have  a  question?

[01:10:54](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4254) Noah Bratzel: Yeah,  you're  gonna  have-

[01:10:55](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4255) Aksana Rahouski: Oh,  sorry,  go  on.

[01:10:56](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4256) Noah Bratzel: Sorry,  one  second.  I  was  just  say,  yeah,  when  you  have  both  in  place,  then  you  have  the  flexibility-

[01:10:59](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4259) Aksana Rahouski: Yes

[01:10:59](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4259) Noah Bratzel: ...  but  you  wouldn't  know,  you  wouldn't  know  for  sure-

[01:11:01](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4261) Aksana Rahouski: Sure

[01:11:01](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4261) Noah Bratzel: ...  until  you  get  through  phase  one,  whether  or  not  you'd  be  comfortable  with  keeping-

[01:11:05](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4265) Aksana Rahouski: Yeah

[01:11:05](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4265) Noah Bratzel: ...  them  in  phase  one.  Yeah,  that  would  be  the  other  point  I  would  make.  It's  like  this  is  all  knowledge  you  would  have  that  you  don't  currently  have.

[01:11:11](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4271) Aksana Rahouski: Absolutely.  Um,  Aaron,  did  you  have  a  question?

[01:11:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4275) Aaron Diefes: No,  like,  the  same  idea.  Just  like,  I  think  it  would  be  extremely  important  to  be  testing,  one,  with,  like,  live  data  and  real,  like-

[01:11:24](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4284) Aksana Rahouski: Yes

[01:11:24](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4284) Aaron Diefes: ...  volume  of  data.  Because  if  we're  talking  about  hundreds  of  thousands  of  devices,  then  we  should  be  testing  every  single  stage  that  we're  developing  on  hundreds  of,  uh,  like,  a  hundred  thousand  devices.  Like-

[01:11:34](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4294) Aksana Rahouski: Yeah

[01:11:34](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4294) Aaron Diefes: ...  that,  I,  like,  I  kinda  see  where,  like,  no  coming  from  that  standpoint.

[01:11:40](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4300) Aksana Rahouski: Yeah.

[01:11:40](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4300) Aaron Diefes: It's  like,  like,  even  if  it  isn't  technically,  like,  live  or  whatever,  like,  we  still,  like,  still  the  way  we  test  it  has  to  be-

[01:11:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4306) Aksana Rahouski: Yes

[01:11:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4306) Aaron Diefes: ...  with  the,  the,  the  exact  data  that's  in  broad,  basically.

[01:11:51](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4311) Aksana Rahouski: And,  and  a  hundred  percent  I'll  align  on  this.  This  is  not  something  you  wanna  go  down  the  rabbit  hole  with  a  bunch  of  assumptions,  right?  This  is  very  important,  in  fact,  to  define  these,  like,  critical  milestones  and  test  it  against  the  actual  data.  Because  results  we  might  be  seeing  on  milestone  one,  and  two,  and  three  might  dramatically  change  the  rest  of  the  roadmap  for  this  thing,  right?  So  hands  down,  like,  a,  a  hundred  percent  agree  on  with,  on  that.  Um,  so  what  do  you  think...  Again,  like,  I  think,  I'm  just  trying  to  figure  out,  like,  what,  uh,  client...  So  the  client  wants  to  kinda  see  the  concept,  right?  And  I  wanted  to  show  it  to  you  guys  just  to  kind  of  get,  like,  initial  reaction  to  it.  And  again,  we  don't  need  to  even  go  just,  like,  this  is  the  final  look  and  feel,  right?  This  is  the  concept  that  we  are  gonna  kind  of  start  going  down  the  road.  I  do  think  that  perhaps  maybe  we  need  to  take  a,  a  moment  here  and  think-...  just  to  have  a  moment  to  pause  and  think,  right?  Digest  all  this  information,  look  at-perhaps  read  this  document  one  more  time,  click  through  these  things,  just  see  what  other  questions,  concerns  w-we're  seeing,  right?  And  maybe  create  this,  like,  very  high-level  roadmap  as  far  as,  like,  how  would  we  tackle  something  like  this?  Because  they  will  have  the  same  questions,  and  I  would  like  to  at  least,  like,  have  a  direction  that  we  can  tell  them  that  here's  how  we're  gonna  be,  uh,  running  through  this  thing,  delivering  A,  B,  C,  D,  validating  along  the  line,  right?  Um,  just  to,  to  give  them...  Well,  to,  and  ourselves,  um,  like,  can  we  do  this?  Can  we  build  something  like  this?  It's  a  lot  of  things  still  need  to  be  fleshed  out.

[01:13:58](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4438) Stone Marballie: So  for  me,  the  sticking  point  right  now  is  where  we're-where  are  we  storing  all  this?  That's  the  part  we're  need  to  continue  thinking  about  it.  So,  like,  when  you  add  a  new  global,  um,  I'm  assuming  we're  gonna  have  a  table  that  stores  the-

[01:14:18](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4458) Aksana Rahouski: Yeah,  all  the-

[01:14:18](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4458) Stone Marballie: configs,  the  config.  So  when  they  add  a  new  w-  if  they  come  in  and  add  a  new  global  config  parameter,  then  we  automatically  add  a  column  to  our  table,  or  how  is  this...  Or  is  our  table  just  a  dynamic,  like  JSON  or  a  blob  or  something  that  we  can  just  always  add  to?  Like,  I  don't  understand.

[01:14:39](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4479) Richard Sacco: That's,  that's  our,  you  know,  job,  is  to  kinda  come  up  with  that  stuff.

[01:14:43](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4483) Aksana Rahouski: Yeah.  Yeah.

[01:14:44](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4484) Richard Sacco: Whether  it's  like  we  wanna  make  it  so  it's,  um,  a  row,  you  add  another  row  in  the  table,  and  you,  you-we  have  all  these  predefined  fields,  the  six  hundred  ninety-four  you  could  select  from,  or  there's  like  a  lot  of  ways  you  could  tackle  it.  But  it's  really  up  to  us  to  decide  what's  the  most,  like,  maintainable  and  just  best  way  forward  in  that.  Yeah.

[01:15:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4508) Aksana Rahouski: Yeah.

[01:15:08](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4508) Stone Marballie: And  then,  and  then  the  ones  that  are  dynamically,  then  we  gotta  attach  those  to  the  device,  right?  Is  that  gonna  be  in  a  link  or  table,  or  is  that  gonna  be  a  direct  migration  to  the  device  table  itself  or...  You  know  what  I  mean?

[01:15:21](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4521) Richard Sacco: Mm-hmm.  There's  a  lot  to  consider  there,  that's  right.  And,  uh,  like  right  now,  there's  just  a  configuration  ID,  and  it's  just  linking  there.  And  there's  also  a  lot  to  consider  with  the  host  name  because  now  there's  really  not  necessarily  a  host  name  per  config.  So  how  are  you  gonna  decide  when  it  checks  in  and  what  to  do?  Uh,  so  there's,  there's  actually  quite  a  bit  of  complexity  here.

[01:15:44](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4544) Aksana Rahouski: Yeah.

[01:15:44](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4544) Richard Sacco: And,  um,  yeah.  Uh,  like  there's  a-maybe  you  wanna  do,  like,  a  MD5  hash  of  all  this  different  config  values  or  something,  I  don't  know,  but...

[01:15:56](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4556) Aksana Rahouski: Uh,  yeah,  there  is  a  lot.  Like  I  just  said,  like,  eh,  in  the  entire,  like,  we  never  even  talk  about  how  would  we,  like...  Let's  assume  we  build  something  like  this,  right?  So  you  can,  okay,  uh,  schema  is  actually  what  I'm  the  least  worried,  right?  Like,  we  can  probably  build  this  pretty  fast.  Um,  layers  is,  like,  next  complexity,  uh,  rules.  I  think  it's  like  the,  the  device,  like,  dynamic  list  building,  considering  this  like  a  rule  engine  that  like,  if,  a,  trying  to  figure  out,  like,  how  do  I  run  through,  like,  a  list  of  questions?  Where  do  I  find  my  data,  right?  Back  to  your  stored  question,  where  do  we  store  that,  right?  Like,  which  value  wins?  How  do  we  show  the  final  set?  And  then,  like,  ap-applying,  right?  How  do  we...  Like,  today,  like,  when  you  hit  Device  Save,  so,  like,  we  can  either  kinda  trigger  it  as  like  a  push  mechanism,  right?  Something  on  the  portal  triggers  device  config  update.  Uh,  we  also  have  this...  Well,  it's  not  like  a,  a  traditional,  like,  pull  approach,  but  it's  triggered  by  a  pull.  They  send  us  kind  of  data,  and  they  request,  "Oh,  maybe  that  tells  us  to,  like,  yeah,  that-give  them  their  configs,"  right?  Like,  if  both  of  these  mechanisms  still  work  and,  like,  especially  the,  the  whole,  like,  check-in,  how  does  check-in  play  into  this?  Does  this  still  work?  So  a  lot  of  things  just  still  kinda,  um,  flash  out.  And,  um,  I'm  just  trying  to  think,  like...  And  perhaps,  like,  you  know,  there  is  some  more,  like,  middle  ground  that  we  can-uh,  like,  but  again,  the  goal  is  to,  like,  not  to  just  take  this  file  and  digitalize  it,  right?  Put  it  in  a  form.  The  goal  is,  like  I  said,  like,  today,  if  they  said,  like,  "Change  one  of  the  global  settings,"  they  literally  have  to  go  every...  change  two  hundred  files  safely,  right?  That  is  the  problem  we're  solving.  Like,  how  do  we  change  s-global  setting  once,  and  we  can  safely  apply  that  to  every  single  device  out  there?

[01:18:13](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4693) Noah Bratzel: But  doing  the  phase  one  that  I  described  before  does  make  it  a  lot  safer.  You  define  the  schema,  you're  not  having  typos,  you're  not  editing  files,  you-

[01:18:21](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4701) Aksana Rahouski: Mm-hmm.

[01:18:21](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4701) Noah Bratzel: It's  like  y-this  is-

[01:18:23](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4703) Aksana Rahouski: Yeah

[01:18:23](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4703) Noah Bratzel: ...  this  is  just-it's  like  this  is  how  you  would  get-  -from  where  we  are  to,  to  there.  And  it's  like-

[01:18:29](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4709) Aksana Rahouski: Yeah

[01:18:30](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4710) Noah Bratzel: ...  I'm  j-I'm  just  trying  to,  like,  paint  that  picture,  so,  like,  the,  uh,  you  add  w-one  piece  at  a  time,  'cause  as  you  add  one  piece  at  a  time,  you  discover.  Like,  if  we  added  just  the,  the,  the  current  logic  we  have,  and  then  we  put  just  the  global  on  top  of  it-

[01:18:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4726) Aksana Rahouski: Mm-hmm

[01:18:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4726) Noah Bratzel: ...  like,  how  much  would  it  simplify?  How  much  would  it,  ho-how,  how  many  things,  you  know...  What  does  it  look  like  then?  And  so  then  you  would  have  an  idea  of,  okay,  do  we  need  as  many  layers  as  we  thought  we  had,  or  maybe  we  need  to,  you  know...  Now,  that  was,  that's  what  I  would-

[01:19:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4740) Aksana Rahouski: Yeah

[01:19:00](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4740) Noah Bratzel: ...  the  reason  why  I'd  like  us  to  do  this  in  phases  is  because  this  plan  sounds  great,  but  it's  unclear  if  we  need  as  mu-as  much  as  we  have....  So  if  we  build  the  parts  we  know  we  need,  so  th-  this  is  where  Agile  comes  in.  You  know,  you  know  you  need  this  much,  so  you  build  that  much.

[01:19:20](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4760) Aksana Rahouski: So  let  me  ask  you  kinda,  uh...  And  again,  I'm  not,  I'm,  I'm  not  denying  that.  Obviously,  uh,  to  build  it  on  one  shot  is  not  the  approach  we  should  be  taking,  and  I'm  not  saying  that.  Um,  we  do  need  to  like,  break  it  into  deliverables  and  validate  along  the  way.  But  what  I'm  like,  I  guess  maybe  I'm  just  like  fuzzy  and  not  seeing  what  you're  saying.  Let's  say  we  just  build  a  schema  and  we  build  just  the  global  layer,  and  then  we  pick  two  devices  to  put  in  this  new  config.  What  does  that  actually  look  like?  Because  let's  say  we  take  one  device  that  has  a  very  specific,  um-  -  and  I  would  take  like,  the  most  complex  one  at  this  point,  because  I  don't  need  you  to  solve  for  simple.  I'll  take  a  device  whose  configuration  I  based  on  I-22  T-Mobile  customer  cord.  Literally,  all  these  things  play  into,  uh,  what  the  final  key  value  set,  like,  uh,  are  we  setting  it  all  in  global?  But  that  means  the  two  devices  need  two  different  sets-

[01:20:33](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4833) Noah Bratzel: No,  it's,  we-

[01:20:33](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4833) Aksana Rahouski: Because  they  are  different.

[01:20:35](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4835) Noah Bratzel: We  start,  we  start  with...  When  we  start  with  phase  one,  we  don't  change  any  of  the  current  logic,  so  it  works  the  way  it  works  with  the  current  mapping,  with  the  model  carrier  plan.

[01:20:44](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4844) Aksana Rahouski: Mm-hmm.

[01:20:44](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4844) Noah Bratzel: And  then  those  all  are  pointing  to  a  template,  which  is  a,  you  know,  digital  s-setup  of  that  file  basically.  It's  replacing  that  file  with  the  schema  base-  basis.  We  set  up  the  schema  the  way  that  it  will  work  at  the  end,  but  we  know  we  have  templates  that  are,  that  are  basically  working  with  the  current  rules  we  have,  and  then  that,  that  maps  to  those  instead.  So  then  as  you  break  it  out,  then  you  say,  now  you  can  actually  look  at  those  and,  and  evaluate,  okay,  how  many  of  these  are  actually  exactly  the  same  between  all  of  the  devices  that  are  using  these  templates?  And  you  can  actually  look  at  that,  and  then  you  could  break  that  out.  And  like,  now,  not  only  do,  is  it  what  the  client  said,  that  these  are  global,  but  we  know  they're  global  because  we  looked  at  it  and  we  evaluated  it.  Boom!  We  move  those  out  of  global,  and  all  these  files  work  the  same.  Now  we  just  add  a  rule  on  top  of  that.

[01:21:36](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4896) Aksana Rahouski: Sure.

[01:21:36](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4896) Noah Bratzel: Existing  rules  that  says,  "Okay,  all  these  we  don't  need  to  match  at  all.  Those  are  just  stuff  that's  global,  and  it  doesn't,  doesn't..."  You  know,  these  take  those  rules  out  of  those  templates,  and  they  come  from  the  global  now.  It's  like,  it's  just  a  matter  of  how,  how  you  implement  it  piece  by  piece.

[01:21:58](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4918) Aksana Rahouski: I  s-  I  st-  like,  I'm  still  like,  um,  not  like  exactly  following,  but  maybe  that's  okay  for  now.  Um,  'cause  I'm  thinking  again,  like,  it,  th-  like  again,  back  to  remember  that  this  file  is  not  physically  attached  to  this  device.  This  file  is  only  attached  to  this  device  because  it  was  mapped  based  on-

[01:22:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4942) Noah Bratzel: Yeah

[01:22:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4942) Aksana Rahouski: ...  layers,  such  what  carrier  it  is-

[01:22:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4945) Noah Bratzel: That's-

[01:22:25](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4945) Aksana Rahouski: -what  device  it  is,  da,  da,  da,  da,  da,  da,  da,  da,  da.  Right?

[01:22:29](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4949) Noah Bratzel: Changing  that  has  to  happen,  but  changing  that  is  the  most  dangerous  thing.  That's  why  I'm  saying  we  change  that  very  gradually  and  slowly,  and  we  keep  the,  the  existing  logic-

[01:22:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4958) Aksana Rahouski: Mm-hmm

[01:22:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4958) Noah Bratzel: ...  and  we  change,  change  what  the  existing  logic  is  pointing  at,  so  then  we  can  safely  change,  then,  the,  the,  add  the  stuff  on  top  of  it,  because-

[01:22:48](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4968) Aksana Rahouski: Sure.  So  you're  saying  just  keep  mapping  mechanism  and  just  really,  uh...  I  mean,  you're  still  saying-

[01:22:55](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4975) Noah Bratzel: Phase  one

[01:22:55](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4975) Aksana Rahouski: ...  at  the  end  of  the  day,  just  take  this  file  and  present  it  as  something  like  this  with  values,  like  a  global,  like  more  like  probably  you  would  want  schema  and  like  one  layer  maybe  only,  right?  And  all  the  values  are  set.  Is  that  what  you're  saying?

[01:23:11](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4991) Noah Bratzel: That's  what  I'm  saying.  Yeah.

[01:23:14](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4994) Aksana Rahouski: Um-

[01:23:15](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=4995) Noah Bratzel: That's  just  phase  one.  You  would  also  have  the  auditing,  so  that  if  anybody  goes  in  and  changes  those-

[01:23:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=5002) Aksana Rahouski: Yes

[01:23:22](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=5002) Noah Bratzel: ...  values  and  those  templates,  I  mean,  it's,  it's  gonna  be  a  similar  pain  as  changing  the  files  because  you're  gonna  have,  like,  two  of  them.  That's  gonna  be  one-to-one  match  in  phase  one.  You're  gonna  have  two  hundred  of  them  that  you'd  have  to  go  through,  but  it's  a  lot  safer  because  you're  picking  existing  templates.

[01:23:38](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=5018) Aksana Rahouski: Mm.

[01:23:39](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=5019) Noah Bratzel: And  I  mean,  not,  I  mean-

[01:23:41](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=5021) Aksana Rahouski: Which-

[01:23:41](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=5021) Noah Bratzel: ...  with  values  that  are  typed,  pre-  presets,  so  you're  not  typing  them  in,  stuff  like  that.

[01:23:46](https://tldv.io/app/meetings/6995fe185725eb0013924e58?t=5026) Aksana Rahouski: Which,  um,  uh,  which  is  gonna  be  really  important  for  us  to  obviously,  like,  track,  change  history,  and  have  an  ability  to  roll  back,  right?  Because  that's  yet  another  thing  to,  like,  think  through.  Like,  how  do  we...  Because  the  user  could  make  a  mistake  for  whatever  reason,  they  should  be  able  to  go  back  to  the  previous,  starting  from  understanding  what  the  previous  was,  right?  And  recognizing  who  made  the  mis-  change  and  such.  Um,  so  how  about  this?  I,  I  think  it's  just,  like,  right  now,  some,  and  I  don't  know  about  you,  but,  like,  I  know  about  me,  on  this  amount  of  information  is  dropped  on  me,  I  almost  need,  like,  a  second  to  just,  like,  think.  And  so  I  want  to  recommend  that  let's  kinda,  let's  stop  here  because  I'm  sure,  like,  each  one  of  you  have  tons  of  questions.  There  are  a  bunch  of  ideas  that  probably  through  each  one  of  your  head  as  far  as  how  could  we  tackle  something  like  this,  where  could  we  simplify  it  and  such.  Um,  maybe  we  do  that,  and  we'll  have,  like,  a  follow-up  where  we'll  just,  like,  part  two,  or  we'll  talk  more  about,  like,  would  we  build  something  like  this?  And  I'm  not  saying  exactly  like  this,  right?  But  w-  we're  basically  shifting  to  a  very  different  concept  here,  right?  Uh,  like,  how  would  we  do  it?  What  are  the  things  that  we  need  to,  like,  worry  about  first,  next,  last?  So  like  you  said,  basically,  like,  I,  I  would  love  to  hear,  like,  each  one  of  you,  like-...  if  you,  if  I  would  ask  you  to  build  something  like  this,  like,  no  budget  restriction,  I  want  this  thing.  I'll  pay  whatever.  How  would  you-  how  would  we  build  this?  Thoughts  on  that?  Why  would  there  be  no  budget?  I  mean,  like,  saying  that  not  to  make  it  a  restriction  for  you  to  think  about,  like,  the  building  part  of  this  thing.  Like,  y-  it's  my  job  to  worry  about  the  budget,  so  I'm  telling  you,  don't  worry  about  it  now.  Like,  I,  I,  I  really  wanna  know,  like,  can  we  even,  like,  deliver  something  like  this?  Yeah,  and  I  read  your  PRD,  it  seems  like  this  is  not  gonna  be  that  hard.  The  only,  the  only  hard  part  is-  Mm  ...  the  transition.  But  building  it  didn't,  didn't  sound  hard  to  me  at  all.  Okay.  Well,  that's,  that's  good  news.  Um,  or  do  you  like...  Okay,  I'm  also  open  to,  like,  how  would  you  want  to--  'cause  again,  like,  from  a  very  kinda  high  level,  like,  I  wanna  know,  "Hey,  guys,  like,  can  you  build  this  for  me?  How  much  it's  gonna  cost  me?"  Mm.  Right?  But  I  also  realized  there  was  a  lot  of,  like,  middle  ground  that  we  need  to,  like,  get  answered,  tons  of  questions,  define  dependency-  Yeah  ...  how  are  we  gonna  build  this?  Da,  da,  da,  da,  da,  da,  da,  da,  da.  Right?  Um,  so  a-  and  I  do,  like,  again,  just  because  I  feel  like,  but  how  much  we'll  just  look  into,  it  might  be  worth  to  kinda--  So  go  think  about  it,  and  let's  resume  maybe,  like,  early  next  week,  and,  like,  follow  up  and  just,  like,  see  as  we  had  a  moment  to  think  and  digest,  like,  what  do  we  think  about  this,  and,  like,  how  would  we  build  something  like  this?  Yeah,  that  sounds  good.  Uh,  so  we're  gonna  touch  back  next  week?  Yeah,  I'll  schedule  something,  'cause  I,  I,  I  think  sometimes,  and  again,  um,  I  don't  want  it  to  be,  like,  owned  by  one  person.  This  is  big,  right?  Mm.  I  think  the  more  heads  we  kinda  put  together  to  think  about,  what  am  I  seeing  that  you  are  not  seeing,  and  vice  versa,  right?  Starting  from,  uh,  w-  what  are  the  things  to  think  about,  to  flash  out?  What  is  the  best  way  to  build  this?  Um,  which,  by  the  way,  like,  I  agree  with  you.  I  hope  you,  like,  no,  you  don't,  like,  I  don't-  I  hope  I  don't  sound  like...  Yes,  we  wanna  obviously  break  it  into  milestones  and  validate  along  the  line,  like,  for  every...  And,  and  I  actually,  we're  in  agree  with  you  with,  like,  how  we  should  approach  it.  It's  just,  like,  I  would  be  curious  to  hear,  w-  what  follows  step  one?  Like,  how  do  we  then  kinda  unpack  the  matching  l-  logic  into  this  new  configurable  matching  logic,  right?  Um,  and  then,  yes,  so  we'll  just  regroup  next  week,  so  you  guys  have  kinda  a  moment  to  think.  And  maybe,  like,  again,  it  might  be  now  with  a  lot  more  context,  right?  It  kinda...  And  I  think  everybody  is  like,  "Let  me  know  if  somebody's  still  kinda  not  clear  on,  like,  the  concept  of  this  thing,"  right?  Um,  to,  like,  pause  and  think  and  maybe  read  the  document  again,  that  perhaps  some  of  the  things  that  didn't  make  sense  before  now  do,  because  we  just  talked  about  this  thing  for,  like,  two  hours,  right?  Thoughts?  Does  it--  Do,  do,  uh,  do,  yeah,  like,  I  don't  wanna  rush  into  this.  I  want  you  to  take  time  and,  like,  get  grounded  in  this.  No,  yeah,  I  think  that's  a  good  idea,  'cause  we  just  went  through  a  lot,  so-  Yeah  ...  we'll  come  back  to  those  next  week.  It's  all  good.  So,  okay.  All  right,  sounds  good.  I'll  catch  you  guys  later  then.  All  right,  perfect.  Cool.  All  right.  Thank  you  guys  so  much.  Yeah.  Bye.  Bye.  See  you.