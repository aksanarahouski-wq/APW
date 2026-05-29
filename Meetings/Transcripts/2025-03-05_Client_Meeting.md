Devon D'Andrea [00:00]: Here,  in-hand  device  is  disconnected  for  seven  days,  it  was  number  one.  And  I  was  trying  to  determine  which  device  would've  been  counting  in  that,  uh,  towards  that.  And  so  what  I  did  was  I  went  to...  Actually,  I  think  I  have  it.  Uh,  yeah.  So  I  took  everything  from  the  browse  devices  screen.  This  was  yesterday.  So  I  did  last  check-in,  newest  to  oldest.  So  I  had  five...  I'm  sorry.  There  was...  Yeah,  so  there  was...  And  I  was  only  looking  at  in-hand  devices.  So  what  it  said  yesterday  was  that  there  were  m-  one  here  and  66  here.

Aksana Rahouski [01:00]: Mm-hmm.

Devon D'Andrea [01:02]: So  what  I  did  was  I  counted  all  of  these  in-hand  devices  that  had  not  checked  in,  that  were  not  checking  in  yesterday,  and  that  was  a  total  of  67.  So  what  I  was  wondering  is  this  one  here,  and  again,  this  was  yesterday,  but  it's  not  doing,  it's  not  reporting  this  way  today.  I  was  wondering,  is  this  one  somehow,  was  this  one  somehow  counting  as  something  that  was  not  on,  that  was  disconnected  for  seven  days?  Which  would  not  make  sense  'cause  it's  not  connected  for  over  a  month.  Hasn't  checked  in  for  over  a  month.  So  I  was  just  curious  as  to  why  yesterday  this  said  one  and  this  said  66,  but  today  it's  zero  and  67,  which  according  to  this  is  100%  accurate.

Aksana Rahouski [01:54]: Hmm.

Devon D'Andrea [01:59]: What  I  don't  really  know  is,  mm,  and  I  might  be  bringing  up  a  whole  another  issue  right  now,  but  in-hand  devices  online  for  greater  than  seven  days,  that  like...  Okay,  so  Right.  So  let's  try  to  understand  what  this  dashboard  is  trying  to  tell  me,  right?  So  is  this  dashboard  telling  me  this,  this  is  the  number  of  in-hand  devices  that  are  online  and  have  been  connected  for  greater  than  seven  days?  Or  is  this  just  showing  me  the  number  of  in-hand  devices  that  had  been  online  for  greater  than  seven  days  when  they  last  checked  in?  And  I  wanna  say  it's  probably  the  latter.

Aksana Rahouski [02:48]: Mm-hmm.

Devon D'Andrea [02:49]: I'd  have  to  look  at  the  uptime  to  see,  uh,  greater  than...  What  is  this?  Greater  than  seven  days.  So  it'd  be  this  one,  this  one,  this  one,  this  one,  this  one,  this  one,  this  one,  this  one,  this  one,  this  one,  this  one,  this  one,  this  one.  And  that  would  be  it.  So  how  many  is  that?

Aksana Rahouski [03:22]: You  have  there  two,  four-

Devon D'Andrea [03:24]: 13,  which  is  the  count  here.  So,  so  what  this  is  showing  is  connect,  u-  last  uptime  is  greater  than  seven  days.  It  doesn't  matter  if  it's  online  right  now,  it's  sh-  showing  how  many  devices  have  uptime  of  greater  than  seven  days  the  last  time  that  they  checked  in.

Aksana Rahouski [03:46]: Mm-hmm.

Devon D'Andrea [03:47]: So  again,  I  mean,  I  don't,  you  know,  it  kinda  goes  to  like,  should  it  be  that  way  or  shouldn't  it?  You  know  what  I  mean?  Like,  I  don't  know  if...  I'd  have  to  think  about  that  to  see  if  like  I'd  want  it  to  be  that  way,  but-

Aksana Rahouski [03:59]: Yeah.

Devon D'Andrea [04:00]: Yeah.  The  real  issue  was  before  I  moved  this  card  over,  because  this,  this  has  nothing  to  do  with  the  card  you  guys  working  on-

Aksana Rahouski [04:06]: Yeah,  yeah

Devon D'Andrea [04:07]: ...  was  why  this  was  showing  one  yesterday,  but  I  don't  know.  We  could  just  maybe  chalk  it  up  to-

Aksana Rahouski [04:17]: Unless  that  one,  like,  today  something  changed  about  that  device.  Did  it  like  come  back?

Devon D'Andrea [04:22]: No,  the  same,  it's  got  the  same...  So  yesterday,  the  last  check-in  was  February  2nd.

Aksana Rahouski [04:29]: Mm.

Devon D'Andrea [04:29]: If  I  go  back  over  here...

Aksana Rahouski [04:35]: 'Cause  I  don't  think,  like,  I  don't  believe  we  pushed  anything  code-wise  yesterday.

Richard Sacco [04:42]: Uh,  because  I  think,  well,  it,  it  could  be  that,  you  know,  it's  a  30-day-

Aksana Rahouski [04:49]: Query  that  I'm  guessing  is  causing  it,  so  calculation  itself  somehow.

Richard Sacco [04:54]: Well,  like,  i-  isn't  that  30  days  though?  Like  March...  Wait,  okay.  Yesterday's  March  4th,  and  then  you  said  it  last  checked  in  when?  The  22nd  or  the  2nd?

Devon D'Andrea [05:05]: Yes.  So  the-

Richard Sacco [05:06]: So-

Devon D'Andrea [05:06]: ...  the  issue  is,  so  the,  the  card,  I  c-  I  created  this  card  because-

Richard Sacco [05:11]: Yeah,  yeah

Devon D'Andrea [05:11]: ...  in  production-

Aksana Rahouski [05:13]: Mm-hmm

Devon D'Andrea [05:13]: ...  you've  got  zero,  and  like,  a-  and  I  guess,  like,  I  don't  know  if  I  put  this  in  the  card,  but  like,  my  question  was  like...  and  I  can't  remember.  It's  been  so  many,  so  long,  you  know,  Richard,  that  we  built  this  particular  part  of  the  dashboard.  Like-

Aksana Rahouski [05:26]: Mm

Devon D'Andrea [05:26]: ...  in  my  mind,  this  would  mean,  since  you  have  this  over  here,  this  would  mean  like  in-hand  devices  that  haven't  been  online  within  the  last  seven  days.  Whereas  this  is  devices  that...  I'm  sorry,  in-hand  devices  that  have  been,  not  been  online  for  at  least  seven  days  but  less  than  30  days,  'cause  this-

Richard Sacco [05:46]: Yeah

Devon D'Andrea [05:47]: ...  should  show  them  how  many  devices  have  not  been  online  for  at  least  30  days.

Aksana Rahouski [05:51]: Mm-hmm.

Richard Sacco [05:51]: So  that's  why  that  one  device  was  there,  was  because  technically  as  of  yesterday  it  was  29  days  because  the  4th  to  the  2nd  is  not  30.

Aksana Rahouski [06:00]: Got  it.

Richard Sacco [06:00]: Because  March-

Devon D'Andrea [06:01]: That  makes  sense

Richard Sacco [06:01]: ...  uh,  February  only  has  got  28  days.

Devon D'Andrea [06:04]: Oh,  is  that  30  days?

Adam Curcie [06:07]: Well,  it  says  30  days

Devon D'Andrea [06:10]: Yeah.  Yeah,  you're  right.  I  guess  that  is  right.  I  guess  that  is  right-

Adam Curcie [06:13]: It's-

Devon D'Andrea [06:13]: ...  'cause  it  was  the-

Adam Curcie [06:14]: I  think  that's  what  Richard  was  about  to  say

Devon D'Andrea [06:17]: ...  the  February

Adam Curcie [06:17]: ...  and  I  think  it's  just  because  it's  February,  which  is-

Devon D'Andrea [06:20]: 'Cause  it's  February,  it's  30  days.  All  right.  All  right.  All  right.  All  right.  All  right.  Um-

Adam Curcie [06:23]: And  only,  only  us  losers  who  were  born  on  the  28th  of  February  keep  that  fact,  you  know,  just  ready-

Devon D'Andrea [06:30]: Yeah

Adam Curcie [06:30]: ...  right?

Devon D'Andrea [06:31]: Yeah,  Adam's  birthday-

Adam Curcie [06:32]: 28  days

Devon D'Andrea [06:33]: ...  is  the  28th.  All  right.  Well,  that's  going  over  here.  Um,  but  Adam,  the  one  thing,  and  we  don't  have  to  really  harp  on  this  too  much,  is  like  do  we  wanna  explore  this  idea  that  like  on  this-

Adam Curcie [06:43]: Like-

Devon D'Andrea [06:43]: ...  dashboard  this  here  represents  any,  any  last  check-in  where  the  uptime  was  greater  than  seven  days,  regardless  of  it's,  if  it's  online  right  now  or  not?  Should  we  just  leave  it  alone,  'cause  we  want  Axon  to  build  us  a  new  dashboard  anyway?

Adam Curcie [07:08]: Well,  it's  supposed  to  be  for  stuff  that's  online,  right?  'Cause  like  the-

Devon D'Andrea [07:12]: You...  Well,  I  mean-

Adam Curcie [07:13]: Hey,  listen,  I'll,  I'll  just-

Aksana Rahouski [07:15]: I  think,  um-

Adam Curcie [07:16]: I  don't  know.  I'm  sorry  I  was  late.  My,  my  whole  day  has  been  really  all  over  messed  up,  but  the,  the,  like  the  high  level,  if  I  recall  correctly,  and  we're  going  back  three  years,  so  that's  not  always  like  my  strongest  memory  capability,  but-

Devon D'Andrea [07:30]: Yeah

Adam Curcie [07:30]: ...  like  the  reason  we  wanted  to  have  that  show  stuff  greater  than  seven  days  was  basically  to  like,  you  know,  just  kinda  tout  like,  "Wow,  our  devices  stay  online."  Right?  And  then-

Devon D'Andrea [07:40]: Yeah

Adam Curcie [07:40]: ...  like  the  stuff  that's  disconnected,  you  know,  for  seven  days  or  more,  like  that's  stuff  like  maybe  customers  need  to  like  figure  out,  and  then  stuff  that's  disconnected  for  over  30  days  is  stuff  maybe  customers  wanna  turn  off.

Devon D'Andrea [07:55]: Right.  Exactly,  and  that  would  give  them  the  opportunity  to  go  in  and  like  in  the  future  maybe  this  is  clickable-

Adam Curcie [08:01]: Yeah

Devon D'Andrea [08:01]: ...  and  then  they  can  get  a  list  of  all  of  these  devices,  and  they  can  like,  "Hey,  I  think  I'll  turn  these  off."

Adam Curcie [08:05]: Or  at  least  some  of  them,  but  y-  you  know.  I  know  that  like  there's,  there's  a  lot,  you  know,  uh,  different  stuff  we  can  do  with  this  dashboard,  so  I  mean,  we  don't  have  to  harp  on  it.

Devon D'Andrea [08:15]: I  would  say  for  now,  I  would  say  just  for  the  sake  of  the  fact  that  we're  not  even  here  to  talk  about  this  today-

Adam Curcie [08:21]: This-

Devon D'Andrea [08:21]: ...  let's  just  move  on.

Aksana Rahouski [08:23]: Yeah,  'cause  I,  I...  Like  again,  the  kinda  greater  question,  right?  To  me,  totally  makes  sense  like  once  you've  kinda  shown  the  numbers-

Devon D'Andrea [08:31]: Yeah

Aksana Rahouski [08:31]: ...  once  you  see  what  they  are,  right?

Devon D'Andrea [08:33]: Yeah.

Aksana Rahouski [08:33]: Like  some  kind  of  pop-up  or  a  dedicated  page  that  can  basically  filter  down  to  these  devices-

Devon D'Andrea [08:39]: Right

Aksana Rahouski [08:40]: ...  whatever  you  wanna  do  with  them.

Devon D'Andrea [08:42]: Right.

Aksana Rahouski [08:43]: Ugh.  Okay.

Devon D'Andrea [08:45]: Okay.

Aksana Rahouski [08:45]: Um,  hey,  can  I  actually,  um...  Do,  do  you  mind  sharing?  'Cause  I  wanna  talk  about,  um,  logs.  One  of  the  tickets-

Devon D'Andrea [08:53]: Logs.

Aksana Rahouski [08:53]: Yes.  So  one  of  the  tickets  that  we  have  in  the  queue  right  now  is,  mm,  device  efficiency  logs,  which  I  know  you  talked  about  this  device  mo-  modification  t-  log  types,  right?

Devon D'Andrea [09:09]: Mm-hmm.

Aksana Rahouski [09:10]: And  I  think,  again,  this  is  where,  um,  my  understanding,  and  this  is  where  like  Richard  maybe  can  like  help  me  here.  Like,  we  have  basically  like  a  system-generated  logs  and  a  user-triggered  logs.  Um,  so  these  are  like  one  of  the  ones  that  are  like  system  generated.  In  my  understanding,  like  this  happens  like  every  time  device  confi-  like  config  gets  pushed-

Devon D'Andrea [09:30]: Mm-hmm

Aksana Rahouski [09:30]: ...  we  create  a  log,  and  so  we  then  like  reveal  these  logs  to  you.  Uh,  administrative  logging  area  or  module,  right,  um,  is  obviously  like  for  you  we're  kinda  handpicking  which  logs  you  guys  need  to  see.

Devon D'Andrea [09:45]: Right.

Aksana Rahouski [09:46]: Um,  so  let's  maybe  talk  a  little  bit  more  on  this  one  as  far  as...  'Cause  I  know  you  just  created,  "Hey,  do  we  wanna  see  these  logs?"  And  again,  we  can  just  yank  these  out,  uh,  so  you  don't  see  just  these,  but  I'm  curious  kinda,  again,  let's  zoom  out  of  this  a  little  and  understand  how  do  you  guys  use  and  for  what  purposes  these  l-  logs?  Like  what  kinda  questions  we  try  to  answer  when  you...  'Cause  these  logs  that  you're  looking  at  right  now,  they're  quite  useless.  Like  which  exactly  what  you  pointed  out,  right?  Um-

Devon D'Andrea [10:17]: Yeah.

Adam Curcie [10:18]: Well,  some  of  them  are.

Devon D'Andrea [10:19]: Some  of  them  are,  but  some  of  them  aren't.

Adam Curcie [10:21]: And  we're-

Devon D'Andrea [10:21]: So  like-

Adam Curcie [10:21]: Yeah,  they're...  Yeah.

Devon D'Andrea [10:23]: So  like,  I  mean,  we  can  kinda  just  go  down  the  list  here,  like  device  changing  from  offline  to  null,  that  means  that  it's  changing  from  offline  to...  That  means  that  it  checked  in  basically.

Aksana Rahouski [10:35]: Mm-hmm.

Devon D'Andrea [10:35]: Right?

Adam Curcie [10:36]: Yeah.

Devon D'Andrea [10:36]: So-

Aksana Rahouski [10:37]: Mm-hmm.

Devon D'Andrea [10:38]: You  know,  do  I  need  that?  I  don't  think  I  need  that,  because  I  already  have  a  check-in  log  for  a  device.  So  if  I  was  wa-  if  I  wanted  to  know  when  this  device  checked  in,  I  have  a  check-ins  ex-  export.

Aksana Rahouski [10:51]: Mm-hmm.

Devon D'Andrea [10:51]: So  like-

Aksana Rahouski [10:52]: Mm

Devon D'Andrea [10:52]: ...  that's  not  needed,  right,  Adam?

Adam Curcie [10:56]: Yeah,  I,  I  agree  with  that.

Devon D'Andrea [10:59]: Um,  you  know,  this  right  here,  Colorado  host  name  changed  'cause  it  got  a  new  config.  That  we  see  in  the  export  logs.  TID  changed  from  null  to...  That...  Ugh.

Adam Curcie [11:13]: That...  Yeah,  I  mean,  that  would  also  be  in  the  export,  you  know,  so  we  might-

Devon D'Andrea [11:16]: That  would  also  be  in  the  export.

Adam Curcie [11:18]: Yeah,  when  we  go  to  that  device  and  download  the  check-ins,  we'll  see  that  the  check-in  before  this  one  had  no  TID-

Devon D'Andrea [11:25]: Yeah

Adam Curcie [11:26]: ...  now  this  check-in  has  a  TID.

Aksana Rahouski [11:28]: Mm-hmm.

Devon D'Andrea [11:29]: Yeah.  Same  here.

Adam Curcie [11:30]: That's  all  it  was  there  to-

Devon D'Andrea [11:31]: Same  here.  Um,  that's  probably  a  device  that  we  turned  on  in  our  office  for  testing  'cause  there's  no  company.  Offline  to  null.  Password  rotated.  That's  a  SysTech  device.  Do  not  need  that,  hard  no.

Adam Curcie [11:45]: Yeah,  I,  I  can't,  I  can't  remember.  We,  we...  You  know,  there  was...  Ugh,  God,  you  wanna  really  talk  about  stuff-

Aksana Rahouski [11:54]: It  was  a  long  time  ago

Adam Curcie [11:54]: ...  you  wanna  talk  about  stuff  that  we're  not  here  to  talk  about

Aksana Rahouski [11:58]: It's  great  to  talk  about  that  stuff.  It  gives  us,  like,  more  visibility,  yeah.  And  that's  where  I...  Again,  sometimes  when  you  guys  bring  up  these  things  that  are  perhaps  irrelevant,  it,  it  does  though  tell  us  these  two,  two  things  are  the  same  thing  really.

Devon D'Andrea [12:13]: Right.

Aksana Rahouski [12:14]: Right?  I...  Instead  of  kind  of  chasing  the,  the,  the  side  effects  of  something,  right-

Devon D'Andrea [12:20]: Mm-hmm

Aksana Rahouski [12:20]: ...  then  it  helps  us  to  identify,  well,  what's  causing  it,  right?  And  that's  why  I'm  asking,  like,  perhaps,  uh...  'Cause  you,  you,  you  submitted  that  ticket,  you  had  very  specific  test  case  where  you  were  like,  "I  was  looking  for  this  device  logs,  and  I  see  pages  and  pages  of  this  configs.  I'm  drowning.  It's  not  helpful."  Right?  So  I'm  just  trying  to  figure  out,  okay,  from  a  perspective  of,  like,  logging,  right?  And  there  are-

Devon D'Andrea [12:42]: Yeah

Aksana Rahouski [12:42]: ...  different  types  of  logs.  Like,  um,  what,  like,  like,  what  are  the,  the,  the  main  cases  you...  And,  and  it,  it,  it  may  be  that  keeping  it  even,  like,  more  narrow,  um,  the  ticket  you,  you  submitted,  you  were  looking  for  a  particular  device  logs.

Devon D'Andrea [12:59]: Yeah.

Aksana Rahouski [12:59]: What  exactly,  uh,  were  you  chasing  in  these  logs  for  that  device?

Devon D'Andrea [13:03]: More  so,  it-

Adam Curcie [13:04]: Well,  yeah

Devon D'Andrea [13:05]: ...  honestly,  it's  more  so  user-initiated  things.

Aksana Rahouski [13:08]: Okay.

Adam Curcie [13:08]: That's...  Yeah,  and  that's  really,  like,  you  know,  you  guys  kinda  in  the  beginning  of  the  sentences,  you  said,  like,  couple  minutes  ago,  you,  you  kinda  like  categorized,  like,  you  know,  stuff  that,  like,  the  system  automatically  generates.  Like-

Aksana Rahouski [13:26]: Yes

Adam Curcie [13:26]: ...  you  know,  all,  all  the  behind  the  scenes  kinda  like-

Aksana Rahouski [13:29]: Yeah,  the  plumbing-

Adam Curcie [13:30]: Like-

Aksana Rahouski [13:30]: ...  I  call  it,  yes.

Adam Curcie [13:31]: Yeah.  Like,  that's  the  stuff...  I,  I'm  almost  wondering  if  it  would  just  be,  mm,  I  don't  know,  logical  to,  like,  split  them.  'Cause  yeah-

Aksana Rahouski [13:39]: Okay

Adam Curcie [13:39]: ...  to  Devin's  point,  when  we  are  trying  to-

Devon D'Andrea [13:42]: I'm  looking  at  a-

Adam Curcie [13:43]: ...  figure  out  stuff-

Devon D'Andrea [13:43]: Yeah,  I'm  looking  at  100...  Just  for  example,  I'm  looking  at  100  right  now,  and  it  goes  back  15  minutes,  right?

Aksana Rahouski [13:48]: Mm-hmm.

Devon D'Andrea [13:48]: So  on  this  page,  on  the  last  100  log  entries-

Aksana Rahouski [13:53]: Mm-hmm

Devon D'Andrea [13:53]: ...  I  mean,  I  would  tell  you  that  probably  the  only  things  that  look  meaningful  to  me  in  any  way  are-

Aksana Rahouski [14:01]: Okay

Devon D'Andrea [14:01]: ...  use...  where  there's  an  actual  user.

Aksana Rahouski [14:03]: Yes.  Yes.

Devon D'Andrea [14:04]: Um-

Aksana Rahouski [14:04]: And  that's  why  at,  at  a  kinda,  a  broad  stroke  rule  of  thumb,  every  time  you'll  see  a  log  that  has  N/A  for  a  user  or  IP  address-

Devon D'Andrea [14:14]: Mm-hmm

Aksana Rahouski [14:14]: ...  it  tells  most  likely  is  system-generated.  So  again,  like-

Devon D'Andrea [14:19]: Yeah

Aksana Rahouski [14:19]: ...  actions  that  are  triggered  by  user,  user  did  something,  we  log  the  action,  right?  That-

Devon D'Andrea [14:24]: Yeah

Aksana Rahouski [14:25]: ...  my  guess  is  that's  what  you're  mostly  after,  and  there  are  tons  of  logs  we're  adding  for,  like,  supportability,  right,  and  traceability,  uh,  uh,  purpose.  So  these  are  all,  like,  every  time  system,  like,  kinda  under  the,  the,  the  ground,  right,  makes  a  decision  to  go  from  A  to  B  to  C,  and  something  changes,  we  log,  log,  log  so  we  can  use  it  for  troubleshooting.  But  to  you,  most  likely,  these  logs  are  just  noise,  right?

Devon D'Andrea [14:50]: Right.  Sorry,  I  was  highlighting  this  just  to  more...  just  to  show  Adam-

Aksana Rahouski [14:56]: That's  okay

Devon D'Andrea [14:56]: ...  something  that  we  probably  should  look  into.  But,  but,  no,  again,  same.  Like,  I,  I,  I  agree,  uh,  100%.

Aksana Rahouski [15:03]: So  we  could-

Adam Curcie [15:04]: Yeah

Aksana Rahouski [15:04]: ...  again,  if  we  don't  wanna,  like,  fully  remove  them,  I  think,  like  Adam  was  just  suggested,  we  could  separate  the  two.  We  could  kinda  give  your...  You,  if  you  ever  need  to  dig  through  system  logs,  you  have  them,  but,  like,  you  have,  like,  more  user-triggered,  um,  action  logs.

Devon D'Andrea [15:22]: Right.

Aksana Rahouski [15:23]: Um,  that's  primarily  probably  why  what  you're  looking  for.

Devon D'Andrea [15:27]: I  would  agree.

Adam Curcie [15:28]: Yeah,  for,  like,  typically,  like,  just  the  other  day  when  we  had  that  issue  with  the  Mealy  devices-

Aksana Rahouski [15:34]: Mm-hmm

Adam Curcie [15:35]: ...  and  we  were  trying  to  figure  out  what  they  specifically  did,  I  mean,  you  know,  it,  it  was  something  that  one  of  their  employees  did.  Um,  a-  and  I  think  they  either  did  it,  like,  incorrectly  or  inadvertently.

Aksana Rahouski [15:48]: Um-

Adam Curcie [15:48]: But  we  were  able  to  basically  determine  this  happened  because  this  person  clicked  this  action.  And-

Devon D'Andrea [15:56]: And  there's  no  better  feeling  than  being  able  to  say-

Aksana Rahouski [15:59]: Yes

Devon D'Andrea [15:59]: ...  we  know  who  exa-  exactly  who  did  it.

Aksana Rahouski [16:02]: Yep.

Devon D'Andrea [16:02]: And  you  caught  red-handed.

Aksana Rahouski [16:03]: That's...  Yeah,  that's  the  main  purpose  of  logs  and  for  you  to  have  access  to  them,  right?  So  if  clie-  client  comes  kicking  and  screaming,  you  can  go  back  and  say-

Devon D'Andrea [16:12]: Right

Aksana Rahouski [16:12]: ...  "Hey,  no,  actually-"

Devon D'Andrea [16:14]: Yeah

Aksana Rahouski [16:14]: "...  because  you  messed  up."

Devon D'Andrea [16:17]: Yes.  Yes.

Aksana Rahouski [16:18]: Okay.  Okay,  so  maybe,  like,  again,  then  we  just-

Adam Curcie [16:22]: However,  I  do  wanna  just  quickly,  like...  'Cause  in  the  original  ticket,  and  I  apologize,  my,  uh,  my  laptop's  having  some  kinda  issue  right  now,  so  I'm  doing  this  all  on  my  phone.  Um,  but  in  the  original  ticket  that  I...  when  I  presented  this-

Aksana Rahouski [16:38]: Mm-hmm

Adam Curcie [16:38]: ...  that  device  that  I  was  searching  for  in  the  logs  gave  me,  like,  literal  pages  of  what  I'm  going  to  refer  to  as,  like,  re-indexing.  It  was-

Aksana Rahouski [16:52]: Yeah

Adam Curcie [16:52]: ...  and  it  was,  like,  just...  And  it  looked  like  it  was  almost,  like,  generating  logs  or  indexes  of  its  own  logging  or  indexing.

Aksana Rahouski [17:04]: Yeah.

Adam Curcie [17:04]: And  I  wasn't...  And  I  was  just,  like,  I'm  like,  "Man,  this...  I  don't  know  what's  going  on,  but,  like,  there  shouldn't  be  this  much  data  wasted  on  whatever  this  is."  But-

Aksana Rahouski [17:15]: Mm-hmm

Adam Curcie [17:16]: ...  uh,  I  don't  know  if  you  have  the,  the  ticket  open  or,  like-

Aksana Rahouski [17:20]: Yeah

Adam Curcie [17:20]: ...  did  we  ever  actually  figure  out  what  that  was  that  was  just,  like,  flooding  in  that  one  device's  audit  log?

Aksana Rahouski [17:27]: Um,  yeah.  Let's  look.

Devon D'Andrea [17:27]: Adam,  where's  the  ticket?

Aksana Rahouski [17:29]: It's  W77193.  That's  the  device.  So  if  you  want-

Devon D'Andrea [17:34]: W77193.

Aksana Rahouski [17:40]: Yes.

Devon D'Andrea [17:42]: What,  all  the  latitude  changes?

Adam Curcie [17:44]: No,  no,  no.  You  gotta,  like,  probably  go  back  a  few  pages-

Devon D'Andrea [17:47]: Well,  this  is-

Adam Curcie [17:48]: ...  'cause  this  was  a  while  ago

Devon D'Andrea [17:49]: ...  January.  I'm  in  January  here.

Aksana Rahouski [17:53]: Well,  I  guess  let's  see  when  this  ticket  was  submitted.  We  can  see  it.  It  was  created...  Oh,  man,  it  was  created  in  August  2015.

Devon D'Andrea [18:01]: Oh,  Jesus.

Adam Curcie [18:03]: Yeah,  you  gotta  go  back  a  few  pages.

Devon D'Andrea [18:06]: I  thought  you  said  that  was  the  recent  Amelie  issue.

Aksana Rahouski [18:09]: I  think-

Adam Curcie [18:09]: No,  no,  no,  no,  no.  That,  that,  that  right  there.  There  it  is.  It's  just  the  log,  device  configuration  log  ID  changed,  and  it  just...  It  w-  Like,  there's  pages  of  it.

Aksana Rahouski [18:19]: Yeah.

Adam Curcie [18:20]: Pages  and  pages.

Devon D'Andrea [18:21]: Why  would  it  be  changing?  Why  would  it  be  changing  that  often?

Adam Curcie [18:23]: Well,  it...  Well,  it's  a  log  ID.  Like,  and  that's  where  I  was  like...  I  was  confused.  I'm  like,  "What's...  What  is  this  logging  ID  changing?"

Devon D'Andrea [18:33]: So,  so  basically  every  time  a  d-device  configuration  log,  that  means  that  a  con-  an  attempt  was  made  to  change  the  config.  And  any  time  that  happens,  it  does  create  a  new  ID  for  that.  So  it  is  attempting  to  change  the  config.  I'm,  I'm  guessing  this  is  just,  like,  a,  a...  This  device  is  a  tough  customer,  but,  uh,  I  don't  know.  Like,  it's  not  accepting  the  config  very  much  or...

Aksana Rahouski [18:59]: Yeah,  for...  Or,  like,  at  a  time,  as  you  can  see,  that's,  like,  every  two  hours  where  we're  probably  getting  check-in,  an  attempt  put  in  in  the  queue  for  a  config  update.  And  every  two  hours  it's  attempting,  and  every  time,  like  Richard  said,  it  creates  new  ID.  And  then  this  is,  like,  again,  system  as-assigns  new  ID  to  the  device  and  logs,  logs,  logs,  logs.

Devon D'Andrea [19:18]: Right.

Aksana Rahouski [19:20]: So,  like,  why  it  was,  like,  kinda  host,  right,  that  device  back  then,  it's,  like,  separate  question  from  should  we  or  not  see  these  logs.

Devon D'Andrea [19:31]: Right.  I'm  not  even  sure  if  I'm  gonna  be  able  to  get  back  to  the  check-ins,  but  I...  We  don't  need  to  see  that.  I  mean,  I,  I  mean,  like,  unless  you  think  that  we-

Adam Curcie [19:39]: Okay.  Well,  yeah,  you...  No,  no,  they  answered  my  question,  so-

Devon D'Andrea [19:42]: Yeah

Adam Curcie [19:42]: ...  I  don't...

Devon D'Andrea [19:44]: Okay.

Aksana Rahouski [19:45]: Um,  so  just  back  to  logs  then  for  a  second.  Do  we  then  kinda  wanna,  like,  um,  pull  out  system-generated  logs  into  maybe,  like,  a  separate  or,  or  take  them  out  entirely  for,  for  you  to  see?

Devon D'Andrea [20:01]: Um,  I  probably  would  be  okay  with  just  taking  them  out.

Aksana Rahouski [20:06]: And-

Adam Curcie [20:07]: Well,  there...  I,  I  mean,  I  feel  like  there  were  a  handful  of  times  where  we  did  actually  use,  like,  like  Dev.  Like  you,  you  know  there's  been  times  where  we  see,  like,  some...  Especially  with,  when  it,  like...  When  we  have  to  kinda,  like,  follow  the  breadcrumbs  for,  like,  billing  active  changed  from  zero  to  one  or  pending  changed  from,  like,  one  to  zero.

Devon D'Andrea [20:31]: Yeah.

Adam Curcie [20:31]: I  feel  like  some  of  those  have  had  some  use  at  some  times.  It's  just,  like,  it,  it's  really  hard,  like,  to  really,  like...  You  know,  if  I  were  to  say,  "Hey,  send  me  a,  like,  a  list  of  every  single  possible  log,  and  I'll  let  you  know  what  we  want."  Like,  that's  gonna  take  weeks,  so.  Like,  to  actually  read  through-

Aksana Rahouski [20:52]: Yeah

Adam Curcie [20:52]: ...  all  of  the  different-

Aksana Rahouski [20:53]: Yeah

Adam Curcie [20:53]: ...  types  of  things  that  get  logged  and  figure  out  what  could  have  value.

Aksana Rahouski [20:57]: Yeah.

Adam Curcie [20:57]: But  yeah,  I  don't  know.  There,  there  has  been  some.  So,  like,  while  I  know  it's,  like,  tempting  to  just  simplify  our  lives  and  say  taking  that,  I,  I  do  kinda  feel  like  at  least...  I,  I,  I  would  say  maybe-

Aksana Rahouski [21:11]: Mm-hmm

Adam Curcie [21:11]: ...  for,  for  the  time  being  we  just  split  them.  I  don't  know.  Dev,  I  don't  know,  what  do  you  think?  Do  you  feel  like  there's  been  enough  times  that  we've  used  those  system-generated  logs  that  kinda  give  you,  like,  the  behind-the-scenes  peek  into,  like,  what  the  portal's  actually  doing  while  it's,  like,  processing  changes?  'Cause,  like,  I,  I,  I  can't  really  say  for  certain,  like,  which  device,  which  customer  when,  but  I  feel  like  there  has  been  instances  where  it  was  useful.

Devon D'Andrea [21:41]: Perhaps,  um,  it  might  just  get  you  closer  to  figuring  something  out,  like,  more  quickly.  Um-

Adam Curcie [21:48]: Well,  yeah,  I  mean,  listen,  like,  the  big  problem  right  now  that  we  deal  with  is,  A,  these  logs  take,  A,  years  to  load  at  times,  right?

Devon D'Andrea [21:58]: Mm-hmm.

Adam Curcie [21:58]: And,  and,  and  also  that  they're  just-

Devon D'Andrea [22:00]: It-

Adam Curcie [22:00]: They're,  they're  convoluted,  so  I  think  splitting  them  out  to  have,  like,  like,  if  we  had  literally  two  links,  like,  instead  of  just  browse  logs,  you  could  have  browse,  you  know,  user  logs  and  browse  system  logs.  'Cause  90%-

Devon D'Andrea [22:14]: Mm-hmm

Adam Curcie [22:14]: ...  percent  of  the  time  we're  gonna  click  browse  user  logs.

Devon D'Andrea [22:17]: That's  fine.

Adam Curcie [22:17]: And,  and,  and  if  it  loads  faster  and  the  logs  are  less  congested,  I  think  that  really  solves  the  problem.

Devon D'Andrea [22:24]: Yeah.

Adam Curcie [22:24]: But  is,  is  that,  is  that  more  work  on  your  guys  to  have  to,  like,  split  it  all  compared  to,  like-

Aksana Rahouski [22:31]: Yeah.  But  it-

Adam Curcie [22:31]: ...  find  it  all?

Aksana Rahouski [22:32]: Yeah,  so  think  about  it  fr-from,  like,  a  perspective  of,  like,  effort,  right?  Obviously,  like,  leaving  as  is  is  zero  effort.  Um,  taking  them  out  would  be  probably  is  the  next  simpler  approach  because  all  you  do  is  just  when  you  query  your  data,  exclude  foo,  right?

Devon D'Andrea [22:49]: Mm-hmm.

Aksana Rahouski [22:49]: Your  UI  does  not  change,  right?  Um,  the  next  step  up  w-  as  far  as  scope  is,  uh,  exclude  from  here  and  also  add  it  elsewhere  where  I  can  still  find  it,  which  that's  where  you're  talking  kinda  splitting,  building  a  u-  a  user  interface  for  that  new  search,  right?

Devon D'Andrea [23:10]: Mm-hmm.

Aksana Rahouski [23:10]: Uh,  so  a-as  far  as,  like,  effort,  um,  higher  than  just  take  it  out.

Devon D'Andrea [23:17]: Got  it.

Aksana Rahouski [23:18]: Because  again,  you  have  to  kinda  adjust  your  U-  front  end-

Devon D'Andrea [23:22]: Mm-hmm

Aksana Rahouski [23:22]: ...  to,  to,  to  address  that,  right?  Whether  simply,  like,  exclude  these.  It's,  like,  you...  Nothing  changes  in  the  front  end.  You  just  see  less.

Devon D'Andrea [23:32]: Right.

Aksana Rahouski [23:35]: Do  you  guys,  you  guys

Devon D'Andrea [23:36]: Yeah,  I  mean-

Aksana Rahouski [23:36]: ...  still  have  them?

Devon D'Andrea [23:36]: I  mean-

Aksana Rahouski [23:37]: Do  you-

Devon D'Andrea [23:37]: ...  removing,  removing  data  points  is-

Aksana Rahouski [23:40]: Uh

Devon D'Andrea [23:40]: ...  goes  against  everything  I  believe  in,  but  I  just  don't  know  if...  I  don't  know.  I'm,  I  guess,  I,  I...  Listen,  I  mean,  if  we  think  that,  if  we  think  that  there's  some  benefit,  whether,  uh,  you  know,  however  small  it  is  to  having  it,  then  perhaps  we  just  have  you  move  forward  with  separating  it  out.

Aksana Rahouski [24:00]: Okay.  Uh,  as  far  as  kind  of  priority  though,  like,  where...  And  again,  since  we  kinda,  again,  have  multiple  things  now-

Adam Curcie [24:10]: This  is  not  a  huge  priority

Devon D'Andrea [24:11]: Yeah,  not  high

Adam Curcie [24:13]: Nah,  nah

Aksana Rahouski [24:15]: So-

Adam Curcie [24:15]: Whenever  you  can  get  to  it

Aksana Rahouski [24:16]: We  can-  Okay.

Adam Curcie [24:17]: Yeah.

Aksana Rahouski [24:17]: We  man-  we'll  manage.  Okay.

Adam Curcie [24:20]: Yeah.

Aksana Rahouski [24:20]: All  right.  Well,  it's  kind  of  the,  the  ...  Is,  is  it  below  ...  Let  me  see.  Kinda,  'cause  the  way  I  see  it  obviously  we're  starting  now  on  the  Verizon  business  acc-  it  doesn't  like  deprioritize  any  of  it.  Smaller  things  like  exclude,  um,  not  just  email,  like  allowing  to  do  both,  in-  include,  exclude,  a  really  small  feature.  Uh,  then,  uh,  let  me  see,  uh,  what  else  is  there.  We  also  have  an  invoice  template  redesign.  Uh,  and  again,  that's  like  lower  priority,  but  at  some  point.  So  I  guess  when  it  comes  to  invoice  or  this,  like  what  wins?  What's  more  important  to  you,  redesign  invoice  or  clean  up  logs?

Devon D'Andrea [24:58]: Invoice.

Adam Curcie [24:59]: Uh,  for  me  invoice,  which  also  in  my  opinion,  again,  which  is  only  worth  in  this  conversation  two  to  three  pennies,  uh,  I  think  that  would  actually  be  a  lot  less  work  based  on  what  you  just  described  with  splitting  the  logs.  So-

Aksana Rahouski [25:13]: I  ...  Invoice.  Invoice  would  be  a  l-  less,  less  work  I  think.

Adam Curcie [25:16]: And  well,  I  mean,  like  m-  'cause  my,  my  reason  for  putting  that  card  in  was  just  the,  like  the  kind  of  like  actually  just  consolidate  things  to  just  make  like-

Devon D'Andrea [25:27]: Right

Adam Curcie [25:27]: ...  the  most  like  I-  I,  like  I  just  kinda  wanted  it  all  to  be  more  like  compact  and  like,  um-

Aksana Rahouski [25:35]: Mm-hmm.

Adam Curcie [25:35]: Yeah.

Aksana Rahouski [25:36]: Yes.

Adam Curcie [25:36]: Kinda  like,  like  an  Excel  sheet  like  s-  like-

Aksana Rahouski [25:38]: Yes

Adam Curcie [25:38]: ...  look  to  just  the  way  that  the  plans.  And  unfortunately  I  lost  my  beautiful,  beautiful  fricking  template  I  had  designed  with  Excel  and-

Devon D'Andrea [25:47]: Yeah,  you,  you  s-  you  started  making  one,  which  was  getting,  getting  pretty  good  and  then  I  made  one-

Adam Curcie [25:53]: Yeah,  yeah

Devon D'Andrea [25:53]: ...  that  was  really  good  and  I  fucking  lost  it.

Adam Curcie [25:56]: We,  we  made  a  beautiful,  beautiful-  ...  creation  and  can't  find  it  unfortunately.

Aksana Rahouski [26:02]: Like  what?

Devon D'Andrea [26:04]: It  was  basically  we  just  created,  we,  we,  we,  we  organized  what's  on  these  PDFs  in  Excel-

Aksana Rahouski [26:11]: Mm

Devon D'Andrea [26:11]: ...  and  like  just  mocked  it  onto  the  existing,  you  know,  PDF  that  we  have  and  it  just  showed  that  like  you  can  save  a  tremendous  amount  of  space.

Aksana Rahouski [26:20]: Yeah.  Well,  is  it-

Adam Curcie [26:22]: Yeah.

Aksana Rahouski [26:23]: I  have  ...  Um,  let  me  take  over  for  a  second.

Adam Curcie [26:26]: Yeah.

Aksana Rahouski [26:26]: Um,  when  we  talked  about  it,  this  was  what  you  guys  had  on  yours.

Devon D'Andrea [26:32]: Oh,  look.

Adam Curcie [26:32]: Oh,  my.  I-

Devon D'Andrea [26:33]: She  ha-  she  has  it.

Adam Curcie [26:34]: She  has  it.

Devon D'Andrea [26:34]: That's  why  you  don't  have  it,  Devin.

Adam Curcie [26:36]: That's  why.

Devon D'Andrea [26:36]: You  gave  it  to  her.  She  has  it.

Aksana Rahouski [26:39]: I  know.  Actually,  this  is  a  screenshot  from  the  meeting  we  were  present.

Devon D'Andrea [26:41]: From  the  meeting.  Yeah.  Well,  her  internal.

Aksana Rahouski [26:43]: So  I  literally,  I  took-

Devon D'Andrea [26:44]: Her  internal  with  that  right  after.  No  problem

Aksana Rahouski [26:45]: ...  I  took  this  and  I  wrote  requirements  based  on  this  screenshot  that  I  have-

Devon D'Andrea [26:51]: See?

Aksana Rahouski [26:52]: ...  so.

Devon D'Andrea [26:52]: Told  you  we're  in  good  hands,  Adam.

Aksana Rahouski [26:54]: So,  so-

Adam Curcie [26:55]: I  tell  everybody  that  we-

Aksana Rahouski [26:56]: Oh,  my  God.  Sorry.  Go  ahead.

Adam Curcie [26:58]: I'm  sorry.  I  was  just  gonna  say,  I,  I  was  talking  to  a,  a  new  customer  who's  coming  on  board.  We  sent  them,  we're  sending  them  five  boxes  to  try  today.

Aksana Rahouski [27:07]: Yes.  Mm-hmm.

Adam Curcie [27:08]: And  I  was  explaining  to  them  how  awesome  our  software  company  is.

Aksana Rahouski [27:11]: Yeah.

Adam Curcie [27:11]: I  said,  "These  guys  ..."  I,  I,  I  ...  'Cause  y-  you  know,  they  wanna  look  at  APIs,  they're  considering,  you  know,  the  power  cycler  options  and  they  might  want  direct  APIs  to  control  their  power  cyclers.  And  I'm  like,  I'm  like,  "Dude,  don't  worry.  Like  we  have  the  best  software  team."

Aksana Rahouski [27:26]: Mm.

Adam Curcie [27:26]: "So  like  i-  if  you  guys  want  this,  I'll,  I'll,  I'll  get  anything  out  of  them.  They're,  they're  the,  they're  the  best."

Aksana Rahouski [27:33]: They  can  deliver  whatever  s-  you  guys  wanna  s-

Adam Curcie [27:35]: Yeah.

Devon D'Andrea [27:37]: Yeah,  so  like  that  makes  sense,  right?  That  ...

Aksana Rahouski [27:40]: Yeah.  So  that's,  that's  what  ...  I  mean,  the  assumption  is  that's  what  we're  building-

Devon D'Andrea [27:45]: Got  it

Aksana Rahouski [27:45]: ...  as,  as  tickets.

Adam Curcie [27:46]: Yeah.

Devon D'Andrea [27:46]: Yeah.

Adam Curcie [27:46]: And  it's  like  I,  I,  I  feel  like  you  guys  already  have  basically  all  of  the  data,  it's  just  re-

Aksana Rahouski [27:53]: Yes

Adam Curcie [27:53]: ...  redesigning  the,  the  way  it  gets  built  into  an  invoice,  so-

Aksana Rahouski [27:57]: Right.  Just-

Adam Curcie [27:57]: I  feel,  again,  I  don't  have  the  credentials  or  the  cre-  you  know,  to  tell  you  it's  less  work.  But  I  would,  I  would  guess-

Aksana Rahouski [28:05]: Yeah

Adam Curcie [28:05]: ...  it  might  be  a  little  less  than  having  to-

Aksana Rahouski [28:07]: That's-

Adam Curcie [28:07]: ...  redesign  the  way  all  the  logs  are  existing.  I  don't  know.

Aksana Rahouski [28:11]: Yeah.  I,  yes,  I  do  agree  with  that  too.  Okay.  So  then,  okay,  so  let's  just  quickly,  uh,  um,  development.  So  invoice.  Okay,  so  where  does  invitation  fits?  Company  invite  and  the  whole  like  the  thing  that  I  brought  up  yesterday,  remember  that  page  I  shared  with  you  that  we  wanna  build  like  a,  a,  a  space  where  you  can  see  all  your  invites,  all  your  conversion  rates,  all  that  are,  that  is  still  pending,  if  you  can  go  maybe  like  resend  it  or  talk  to  them.  Basically  visibil-  vis-  visibility  into  your,  I  don't  know  what  you  call  it,  but  like,  yeah,  come  in-  invitation  layer.

Adam Curcie [28:49]: Yeah.

Aksana Rahouski [28:51]: Where  does  that  fit  as  far  as  priorities,  um-

Adam Curcie [28:53]: Yeah,  I  mean-

Aksana Rahouski [28:54]: ...  into  logs  and  invoice  and  everything  else?

Adam Curcie [29:00]: I  would  like  that  be-

Aksana Rahouski [29:01]: Just-

Adam Curcie [29:01]: I  would  like  that  before  this  logging  stuff.

Aksana Rahouski [29:04]: Okay.

Adam Curcie [29:05]: Uh-

Aksana Rahouski [29:05]: What  about  invoice?

Adam Curcie [29:07]: Invoice?  Uh,  no,  it  could  be  after  that.

Aksana Rahouski [29:11]: Okay.  All  right.  Okay.  Sounds  good.

Adam Curcie [29:16]: Yeah.  I  mean,  it's  a  really  nice  feature  that's  on-

Devon D'Andrea [29:18]: It's  work,  it's  ...  Yeah.  I  mean,  we,  we  managed-

Adam Curcie [29:20]: Like-

Devon D'Andrea [29:20]: ...  but  like,  you  know,  we-

Adam Curcie [29:22]: Yeah

Devon D'Andrea [29:22]: ...  like  once  we  have  it,  we  realized-

Adam Curcie [29:23]: The  invite  works  well.  It  just  doesn't  like  ...  There's  just  a  few  loose  ends  we  wanna  close  up.  It  do-  but  it  does  work  pretty  well.  I,  I  mean-

Devon D'Andrea [29:31]: Oh,  it  does.

Aksana Rahouski [29:32]: Why  not?

Adam Curcie [29:33]: Yeah.

Devon D'Andrea [29:33]: It's  just,  it's  just  a  blind,  you  know-

Aksana Rahouski [29:35]: Yeah.

Devon D'Andrea [29:35]: ...  it's  just,  it's  just  a  blind.

Aksana Rahouski [29:36]: So  a,  A,  it's  blind,  like  I  said  yesterday,  that  you  do  not  see  how  many  invitations.  And  actually  did  look  at  your  prod  database.  I  think  we're  under  200.  I  don't  know.  Do  you  even  know  like  how  many?  Let  me  quickly  connect  to  product.

Adam Curcie [29:50]: Oh,  no.  We  have  no  clue.  We  just  rifled  them  out.

Devon D'Andrea [29:53]: We  send  them  out.  Yeah.

Aksana Rahouski [29:55]: Y'all  on  there.  At  least-

Adam Curcie [29:57]: Another-

Aksana Rahouski [29:57]: Yeah.

Devon D'Andrea [29:57]: You  get  an  invite.  You  get  an  invite.

Aksana Rahouski [30:01]: Um,  what  we're  missing  though,  like  I  said,  we're  not  tracking  what's,  what  happened  to  these.

Devon D'Andrea [30:06]: No.  No.

Aksana Rahouski [30:07]: All  of  them.

Devon D'Andrea [30:08]: If  Adam,  if  Adam's  like  at  a  trade  show  or  talking  to  some  hot,  you  know,  new  prospect-

Aksana Rahouski [30:13]: Yeah

Devon D'Andrea [30:14]: ...  and  he's  like,  "Oh,  I'm  gonna  invite  you  in,"  and  then,  you  know,  we  lose  track  of-

Aksana Rahouski [30:19]: A  hundred  customers.  It's  like,  oh,  did  we  invite  them?  Yeah.  So  we  have  149  invites  currently  in  production.  Wow.  But  again,  like  what...  That's  more  like  what's  missing.  And  I  do  actually  think  even  though,  like  you  said,  you'll  live  fine  without  it  right  now,  if  you  use  kind  of  this,  like,  invite  approach  to  bring  in  more  customers  and  also  use  your  distributor,  like  m-  like  k-  it's  like  almost  your,  like  your,  your  sales  channel,  right,  too.  Yeah.  Yeah.  Distributors  invite  too.  So  y-  you  are  literally  fly-  A,  flying  blind.  You  don't  know  how  many  are  going  out.  Even  if  somebody  tells  you,  "Yeah,  we're  inviting,"  you  can't  validate  if  they're  inviting  or  not,  right?  Yeah.  And,  and  B,  you  cannot,  you  cannot,  like,  again,  measure  kind  of  the,  the  conversion  rate.  Like,  how  many  of  these  people  actually  click  on,  how  many  come  back,  register  company,  what  companies  registered  from  an  invite.  Sure.  They  give  huge  insights  into  kinda  how  your  sales  work.  So-  Um,  device  group,  device  group  mitch-ma-  mismatch  reporting.  Adam,  does  that  need  to  be  right  there?  'Cause  I  don't  think  it  does.  I  think  that  one  is...  Do  you  know  what  this  one  is?  You-  it-  Well-  Yeah.  It's-  It's,  it's...  Remember  when  we  worked  on,  um-  I,  yeah,  I  know  what  it  is,  but  like-  Yeah.

Adam Curcie [31:36]: Yeah.  It-  it's  a  really,  really  low  number  of  devices  though.

Aksana Rahouski [31:39]: We  can  live  without  that.  Okay.

Adam Curcie [31:41]: Yeah.

Aksana Rahouski [31:42]: So  let's,  like,  deep  pri-  like,  maybe,  like...  I  will  put  it  at  the  very,  very  bottom,  um-  Sure  ...  but  that's,  that's,  that's,  that's  great.  The,  I,  and  I  actually  agree  with  that  too.  I,  I  was  hoping-  Yeah  ...  you'd  say  that  'cause  like  I  know-

Adam Curcie [31:55]: Well,  you  guys-  Well,  hold  on.  Wait.  Like,  so  you  guys  already,  like,  turned  off  the  thing  that  was,  that  I-

Aksana Rahouski [32:02]: Yeah.

Adam Curcie [32:02]: 'Cause  I,  I  wanna  say  the  service  plan  change  is  what  prompted  us  to  learn  about  that,  right?

Aksana Rahouski [32:07]: Yes.  Right.  Yes,  correct.

Adam Curcie [32:08]: And,  and  that's  no  longer  in,  in  motion.  So  what  basically  that  card  is  just,  like,  finding  a  way  to  do  that  same  service  at  a  more,  like,  appropriate  place  in  the  system,  right?

Aksana Rahouski [32:21]: That  card  is  what  you  guys  would...  We  decided  instead  of  kind  of  focusing  on  when  we,  like,  align  these,  we  decided  to  build,  like,  a  report  that  gives  you  the  list  of  them.

Adam Curcie [32:32]: Yeah.  Yeah,  yeah.

Aksana Rahouski [32:33]: So  that-  Yeah  ...  that  would  give  you,  like,  a-

Adam Curcie [32:34]: A  report  that  might,  like,  just  run,  like,  monthly  or  something  instead  of,  like,  just  waiting  for  a  change  on  that,  that  tr-  like,  uh-

Aksana Rahouski [32:43]: So  I  mean,  we  just-

Adam Curcie [32:44]: Is  that,  is  that  what  it  is?

Aksana Rahouski [32:45]: Yeah.  So  we  simplified  it,  that  basically  we  would  put  it  under,  like,  an  admin  section  page,  and  when  you  load  that  page,  it  would  just  load  with  the  page,  and  you  can  export  all  of  them,  and  that  would  be,  like,  a  job  base.  But  basically,  what  it...  It  would  give  you,  like,  um,  a,  a  current  live  status  as  far  as  how  many  devices,  uh,  which  company  they  belong,  which  plan  they're  on,  and  how,  like,  how  is  this  group  not  matched.  Like,  what  is...  Uh,  and  the,  the-

Adam Curcie [33:14]: Okay

Aksana Rahouski [33:14]: ...  these  are  the  ones  where  a  device  group  doesn't  match  my  service  plan  group,  right?  So  we've  decided  when  we  found  it,  we're  like,  instead  of  kind  of  focusing  on  how  do  we  sync  this  up,  right?  Y-  you  just  give  it  to  us.  Give  us  the  list  of  how  many  are  actually  addressed-

Adam Curcie [33:29]: Yeah

Aksana Rahouski [33:29]: ...  like  this.  But  what  is  determining  how  you're  creating  that  list?  It's-

Adam Curcie [33:35]: No,  I  think-

Aksana Rahouski [33:36]: ...  data.  It's,  uh-

Adam Curcie [33:37]: It's  what...  Like,  what's  going  to  trigger  when  it  occurs,  I  guess,  is,  is  I  think  what  Devin  meant  to  ask.

Aksana Rahouski [33:43]: Yeah,  like-  Yes.  And  which-

Adam Curcie [33:45]: Is  it  us  clicking  it,  or  is  it  gonna  run  monthly  or  quarterly  or-

Aksana Rahouski [33:49]: Oh,  that  ri-  d-  what,  what's  gonna  build  the  report  itself?

Adam Curcie [33:53]: Yes.

Aksana Rahouski [33:53]: Like,  what's  gonna  trigger  it  to  query  everything  to  put  the  data  together?  Yes.  So  y-  like  you,  it's  the  same  as  what  you  do  when  you,  um,  browse  jobs  history.  You're  gonna  be  at  page,  and  it's  gonna  load  immediately.

Adam Curcie [34:05]: Ah.

Aksana Rahouski [34:05]: So  when  we  go  to  the  page,  it's  gonna  just  do  this,  the  whole  job.  It's  gonna  query  every  single  box  and-  Well,  it,  it's  gonna  give  you...  We-  we're  gonna  get  all  that  we're  gonna  get.  And  again,  it's  because  we're  looking  at,  uh,  portal  data,  right?  Data  already  tells  us  which  one  are  not  the  same  'cause  we  could  see,  like,  device  you're  on,  service  plan.  Why  do  you  have  the  same  group?  Now  you  make  the  cut,  right?  You  are  on  the  list.  So  this  page-  Okay  ...  on  page  load  give  you  these  right  away.  We  would  paginate  and  export,  like,  e-  because  you  said,  uh,  give  us,  like,  an  ability  to  export  it.  Um,  export-  Yeah  ...  that's  where  really  it  needs  a  job  because  it  needs  to  grab  all  of  these.  Yeah.  Paginating  allows  us  to,  like,  limit  to  X,  which  makes  page  load,  um,  manageable.  So  that's  why  you  kinda-  Okay.  That's,  that's  how  we,  that's  how  we  talked  about  executing  on  this,  the  report.  I'm  not  following.  You  don't  know  what  lives  on  the  carrier  end  as  far  as  device  group  unless  you  asked  for  it.

Adam Curcie [35:09]: That  may  not  be  true.  They  might  already  have  that.

Aksana Rahouski [35:12]: So,  so  what  we  do  is  when  we  change  it  and  it  succeeds,  we  record  it  as  that.  But  if  it  has  changed  outside  of  us-  We  don't  know  ...  we  don't  know.  You're  correct.  Yeah.  So  really-  Yes  ...  this  just  gives  you  a,  the  list  of  what  is  not  aligned,  which  still,  like,  problem  of  why  and  how  do  we  close  the  gap  still  exists,  so  that's  why-

Adam Curcie [35:35]: So  that's...  Yeah,  but  we  don't  really,  I  don't  think  at  this  time  need  you  to  worry  about  that  because  I,  again,  like,  I  think  when  we  looked,  right,  it  was,  like,  well  under  100  devices.  I  don't  even  know.  I  can't  remember  how  many,  but-

Aksana Rahouski [35:48]: Well,  for,  uh,  for,  for,  um,  uh,  ATM  unlimited,  I  think  we  pulled,  like,  a-  around  80,  and  I,  I  don't  know  if  there  are  any  for  other  plans,  but  that  was  the  number  for  that  plan.  Okay.

Adam Curcie [36:03]: Yeah,  I  mean,  we  can  definitely  work  through  it  because  it,  it's,  you  know...  I,  I  wanna,  I  wanna  say  that  that,  whatever  number  that  was,  if  it  was  above  100  or  below  100-

Aksana Rahouski [36:14]: Mm-hmm

Adam Curcie [36:14]: ...  that  was  the,  that  was,  like,  a  product  of,  I  would  guess,  multiple  years.  Mm,  my  guess  is  possibly  since  the  start.

Aksana Rahouski [36:25]: Yeah.

Adam Curcie [36:26]: Um,  like  of  all  the  times  we  had  to  manually  change-

Aksana Rahouski [36:30]: Yeah

Adam Curcie [36:30]: ...  a  device  group,  like  in-  independently  from  the  portal  knowing,  uh-

Aksana Rahouski [36:36]: Mm-hmm

Adam Curcie [36:36]: ...  which  is  at,  which  is  at  times  actually  necessary  for  better  or  for  worse,  'cause  it  keeps  the  billing  correct  when  the  data  doesn't,  um,  fit,  and,  and  that  does,  that  has  happened  a  bunch,  unfortunately.  So,  like  I  said,  you  know,  if  we're  doing  this  on  a  more  frequent  basis,  it's  gonna  be  very  few.  So  you  guys  probably,  at  least  for  the  time  being,  I,  I  would  say  certainly  don't  need  to  worry  about  figuring  out  the  why,  'cause  we  probably-

Aksana Rahouski [37:01]: Okay

Adam Curcie [37:01]: ...  already  know  it.

Aksana Rahouski [37:02]: Okay.

Adam Curcie [37:02]: So  it's  really  not  about  why  it's  happening,  it's  just  good  to  have  visibility  over  ones  that  we  need  to  address.

Aksana Rahouski [37:08]: Okay.

Adam Curcie [37:08]: And  it  should  be  a  relatively  workable  number  that's  not  gonna  tie  anybody  up  for  more  than  20,  30  minutes.

Aksana Rahouski [37:15]: Mm-hmm.

Adam Curcie [37:15]: Um,  but  yeah.  I  mean,  I  guess  right...  So  in  my  head,  the  way  I  think  about  this  is  we  come  into  this  page,  and  do  we  need  a  button  that  basically  is  gonna  query-

Aksana Rahouski [37:28]: Mm-hmm

Adam Curcie [37:28]: ...  every  single  ICC  ID  to  every  single  respective  carrier  to  make  sure-

Aksana Rahouski [37:34]: Oh,  so  to  validate

Adam Curcie [37:35]: ...  'cause,  'cause  that's-

Aksana Rahouski [37:36]: Yeah.

Adam Curcie [37:36]: Yeah,  'cause  to  like,  you  know,  to  Richard's  point,  if  it  succeeds-

Aksana Rahouski [37:40]: Yeah

Adam Curcie [37:40]: ...  it's  logged,  but  we  don't  actually  know  if  every  single  one  is  what  it's  supposed  to  be.

Aksana Rahouski [37:47]: I...  Yeah,  and  I  would  even  like,  I  guess,  ask  a  question  to  take  it  even  like  a  step  further.  Should  we  not  only  ask,  but  a-  attempt  to  patch  it?

Adam Curcie [38:01]: No.

Aksana Rahouski [38:03]: No?

Adam Curcie [38:03]: No,  you  shouldn't  attempt  to  patch  it.

Aksana Rahouski [38:05]: Okay.

Adam Curcie [38:05]: Because  there's  instances  where  it  does  need  to  temporarily  m-  be  mismatched.

Aksana Rahouski [38:10]: Gotcha.

Adam Curcie [38:10]: Unfortunately.

Aksana Rahouski [38:12]: So,  um-

Adam Curcie [38:12]: It's...  Yeah.

Aksana Rahouski [38:13]: Yeah.  So  what  you're  saying,  we  wanna  not  only  kinda  show  the  data,  let's  say  page  load  gives  us  the  list  of  all  the  devices  that  are  mismatched  against  those  service  plan  groups,  um,  and  then  we  need,  um,  a  validation,  like  we  actually  do  one  where  like  h-  these  devices  ping  their  carriers  and  find  out,  is  it  in  fact  A  or  B  or  whatever,  the...  It,  it,  on  the  carrier  side,  if  it's  the  same  value  as  it  is  on  the  portal  side.

Adam Curcie [38:43]: Yeah,  but  my,  my  thought  process  or  concerns  with  like  having  a  button  that's...  we,  we  would  press-

Aksana Rahouski [38:51]: Mm-hmm

Adam Curcie [38:51]: ...  is  that  there  are  limitations.  I  don't  know  them  off  the  top  of  my  head,  but  I  just  know  that  there  are  certain  limitations  that  have  to  do  with  the  number  of  API  calls  per  like  second,  minute,  hour.

Aksana Rahouski [39:05]: Yeah.

Adam Curcie [39:05]: So  I  almost  feel  like  having  this  report  just  generate-

Aksana Rahouski [39:09]: Yeah

Adam Curcie [39:10]: ...  monthly,  and  then  just  have  it  available  to-

Aksana Rahouski [39:13]: Mm

Adam Curcie [39:13]: ...  like  regenerate  if  we  really  like  need  to,  but  like  you  can't...  It  can't  be,  it,  it  can't  be  a  report  that  generates  quickly.

Aksana Rahouski [39:21]: You'll  also  like-

Adam Curcie [39:22]: So  like-

Aksana Rahouski [39:22]: Uh,  you  also  kinda...  To  me,  like  m-  m-  middle  ground,  right?  You  could  still  have  this  page  that  gives  you  kinda  at  least  the,  the  count,  right?  And  like,  like  fuzzy  matching,  like  we  think  this  is  what  the  situation  is.  And  the  button  will...  What,  what  button  will  do  is  actually  kick  off  the  job  that's  gonna  loop  through  this  list,  and  it's  gonna  ping  against  the  carrier  to  validate  and  send  you  a,  like  an  Excel  spreadsheet  as  a  final  result  with  validations.

Adam Curcie [39:52]: Yeah.  I  mean,  like  I,  I,  I  think  that,  you  know,  under  the  kinda  assumption  should  take  at  least  a  few  hours  because  we  can't  send  too  many  API  calls  to  you.  But  I  don't  know  the  rules,  so  maybe  I'm  completely  wrong.  Maybe  you  guys  can  do  it  in  an  hour  or  two,  I  don't  know.  But  I,  I  feel  like  it's  a-

Aksana Rahouski [40:09]: I'd  like  to  just,  I'd  like  to  just  state  that  this  should  be  n-  we  should  be  putting  the  m-  the  smallest,  smallest  amount  of  hours  possible  into  this.  This  is-

Adam Curcie [40:20]: Okay

Aksana Rahouski [40:20]: ...  easily,  this  is  easily  looked  at  from  exporting  throughout  the  various  systems  for  us.

Adam Curcie [40:26]: Okay.

Aksana Rahouski [40:27]: I  just...  I  can't  sit  here  and  think  that  we  should  be  spending,  like  I  don't  even  know-

Adam Curcie [40:32]: Is  it  easy  to  get  a  full  device  export  from  Verizon  still?

Aksana Rahouski [40:36]: Yeah,  it  takes  10  minutes.

Adam Curcie [40:39]: All  right.  Well,  I  mean,  then  we  really  don't  even  need  this  if  you-

Aksana Rahouski [40:43]: That's  kind  of  what  I'm  getting  at.

Adam Curcie [40:45]: Okay.

Aksana Rahouski [40:45]: All  right.

Adam Curcie [40:46]: So  we,  then  we  could  just  m-  I  mean,  block  this  one  for  now  un-  until  we  have  more  briefing.  Well,  just  put,  stick  it  all  the  way  on  the,  the  left  column,  I  guess,  right?

Aksana Rahouski [40:57]: I  see.

Adam Curcie [40:57]: That-

Aksana Rahouski [40:57]: Sorry,  your  eyes.  Okay.

Adam Curcie [40:58]: The...  No,  no,  no.  The,  the  backlog  backlog.

Aksana Rahouski [41:01]: Okay.

Adam Curcie [41:01]: Put  it  on-

Aksana Rahouski [41:01]: Do  we  have  a  column  for  like,  you  know-  A  graveyard  of  like  the  ...  a  graveyard.  Yeah.  Like  we'll  think  about  you  when  we're  ready.

Adam Curcie [41:08]: Yeah.

Aksana Rahouski [41:09]: Uh,  which...  This  is,  yeah.

Adam Curcie [41:10]: That...  Can  we,  can  we  re-  can  we  rename  the,  the  very  first  card?  Can  we  just  rename  it  Last  Chance  Saloon?  Just-

Aksana Rahouski [41:17]: Yeah.

Adam Curcie [41:20]: Maybe.  Maybe  one  day.

Aksana Rahouski [41:22]: All  right.

Adam Curcie [41:22]: All  right.

Aksana Rahouski [41:23]: Let's,  let's  take  it,  this  off  our  plate,  and  then-

Adam Curcie [41:25]: Yeah

Aksana Rahouski [41:26]: ...  focus  on  other  things  that  matter  more.  Um,  okay.  So  let's  see.  So  what  we  have  right  here,  and  configs  are  not  on  here,  and  that's  like  a,  a  big  thing  that's  gonna  be  coming  through  parallel  to  all  these  little  things.  Um,  logs,  we'll  take  a  look.  We'll  pitch  you  a  solution  for  logs,  try  to  come  up  with  something  budget  friendly  there,  but  w-  m-  with  an  assumption  that  you  don't  wanna  lose  any,  you  just  wanna  kinda  separate  them.  Um,  okay.  So  not  to  kinda  like...  I  wanted  to  like  just...  Maybe  we  don't...  Like,  uh,  there  are  a  few  tickets  here  about,  uh,  W9.  So  I  just  wanted  to  kinda  see  the  urgency  for  these,  and  I-  Not  until,  not  until  Q4.  Oh,  okay.  So  maybe  we'll  just  table  it  for  now.  Okay.  Okay.  So  I  think  then  it  gives  us  plenty  for  now.  I  think  maybe  we're  good  as  far  as  these  meetings.  We  have  a  lot  of  work  prioritized,  a  lot  of  work  in  flight,  and  like  I  told  you,  I  want  next  week  to  book  something  where  we  can  start  talking  about  configs  together.

Adam Curcie [42:31]: Yeah.  Um,  I  also  have  Vince  on  standby  for  whenever  you  wanna  start,  um,  kick-

Aksana Rahouski [42:38]: Sure

Adam Curcie [42:38]: ...  kicking  off  the  meeting  about  the,  uh,  the  credit  cards.

Aksana Rahouski [42:42]: Okay.

Devon D'Andrea [42:43]: He'll  be...  I  told  him  he  needs  to  be  a  heavy  participant  in  that.  So-

Aksana Rahouski [42:47]: Okay

Devon D'Andrea [42:48]: ...  um,  just  keep  that,  keep  that  in  mind  because-

Aksana Rahouski [42:51]: Okay.  Yep

Devon D'Andrea [42:51]: ...  he's,  he's  a-  he's  probably  our,  uh,  just  by  an  hours  standpoint,  busiest  co-  uh,  uh,  employee,  so  I  just  gotta  make  sure-

Aksana Rahouski [43:01]: Gotcha

Devon D'Andrea [43:01]: ...  he's  good.

Aksana Rahouski [43:02]: No  worries.

Devon D'Andrea [43:02]: And  the  one  item  that  is  at  the,  uh,  very  top  of  the  prioritized  backlog,  John  from  our  team  added  it,  and  he  put  it  in  the  regular  backlog.  And  I  said,  "No,  no,  no.  That  needs  to  be  looked  at  because  it's  just  something  that  our  customers  can  see."  Um,  is  that  something  that  is  kind  of  in  this  next...  I  guess  you  guys  still,  you  guys  still  have  to  groom  it  actually  it  looks  like.

Aksana Rahouski [43:29]: Um,  are  you  referring  to  the  bug-

Devon D'Andrea [43:31]: This  wasn't  in-

Aksana Rahouski [43:32]: ...  this  T-Mobile?

Devon D'Andrea [43:34]: Yeah.

Aksana Rahouski [43:35]: Yeah.  So  we,  like,  ev-  like,  just  to  kind  of,  you  guys  know  how  we  operate.  Every  time  you  put  something  in  and  you  mark  it  as  a  bug-

Devon D'Andrea [43:42]: Yeah

Aksana Rahouski [43:43]: ...  that  is  our,  like,  top  priority.

Devon D'Andrea [43:45]: Got  it.

Aksana Rahouski [43:45]: Somebody's  immediately  on  it.

Devon D'Andrea [43:47]: Okay.

Aksana Rahouski [43:48]: So,  like,  that's  kind  of  for  you  to  know.  If  you,  if,  if  it's,  like,  a  bug  plus  priority  high,  that's  like-

Devon D'Andrea [43:55]: Mm-hmm

Aksana Rahouski [43:55]: ...  screaming  at  us.

Devon D'Andrea [43:56]: Yeah.

Aksana Rahouski [43:56]: That's  how  we  think  about  it.

Devon D'Andrea [43:58]: Okay.

Aksana Rahouski [43:58]: Uh,  but  any  bug,  it,  like,  deprioritized  everything  and  bubbles  up  at  the  top.

Devon D'Andrea [44:04]: Okay.  Um,  and  do  we-

Aksana Rahouski [44:06]: That's  how  we  try  to  look  at  that  backlog

Devon D'Andrea [44:08]: ...  do  we  need  to  touch  on,  um,  'cause  Adam,  didn't  this  come  up  with  like,  um,  didn't  this  come  up  with,  um,  a  different,  an  I  52  or  something  recently  where,  like,  the  carrier  name  was  showing  up  on  the  check-in,  on  the  last  signal  strength  check-in  in-

Adam Curcie [44:23]: Oh,  I...  Was  it  because,  like,  um-

Devon D'Andrea [44:27]: Because-

Adam Curcie [44:27]: ...  it  was,  it  was  a  T-Mobile,  and  did,  we  didn't  know  if  we  had  everything,  like,  that-

Devon D'Andrea [44:32]: Yeah,  I  wanted  to,  yeah,  maybe-

Adam Curcie [44:33]: 'Cause  I  don't  remember  if  we  ever  even  had  a  conversation-

Devon D'Andrea [44:36]: Yeah

Adam Curcie [44:36]: ...  about  handling  T-Mobile,  Verizon  dual  carrier  devices.

Devon D'Andrea [44:39]: How  to  decipher  between  a  T-Mobile  check-in  versus  an  AT&T  check-in  or  a  Verizon.

Adam Curcie [44:45]: Oh,  wait,  no.  Yeah.  We're,  uh,  but  I  think  it  was,  I  think  it  was  an  AT&T.  I  can't  remember.  I,  you  know  what?  It  was  one  where,  like,  the,  the  Verizon  active  checkbox  was  flagged,  but  it  was  using  the  AT&T.  So  even  though  it  came  in  on  a  10.64  IP  address,  the  portal  showed  it  as  a  Verizon.  Is  that  what  it  was?

Devon D'Andrea [45:08]: Is  that  what  it  was?  I  don't  know.  I'm  not  quite  sure,  but  can  we  have  a  quick  discussion  if  possible?

Aksana Rahouski [45:12]: Yeah,  yeah.  Let's,  um,  let's  talk  about  this  one.  Do,  do  you  mind  sharing  and  show  us  what  you're,  like,  seeing  and...  'Cause  we,  I  don't  think  anybody  actually  jumped  on  that  bug  yet,  so.

Devon D'Andrea [45:20]: No,  no,  that's  fine.  Yeah,  I'll  share.

Aksana Rahouski [45:23]: Yeah.

Devon D'Andrea [45:23]: So,  and  again,  we,  you  know,  we  were  looking  at  something  last  week  that  was  similar,  but  we  don't  necessarily  know  exactly  what  it  was.  But  for  this  ticket,  we're  looking  at...  So  these  are,  um,  so  these  are  ATM  link  origin.  Let's  see  if  I  can...  60046000760.  I  can  remember  that.  46000767.  Well,  that's  a  bad  example.  Um,  these  might  be  one  of,  these  might  be  a  couple  ones  that  we  changed  from  T-Mobile,  but  0460001040.

Aksana Rahouski [46:25]: Aaron,  do  you,  Aaron,  do  you  wanna  jump  in?  Aaron's  just,  like,  pinging  me,  and  I  want  him...  Like,  Aaron's  been  doing  a  really  good  job  on  this  project,  so  I  wanna  make  sure  he  gets  the  chance  to  shine  here  too.  Noah  looked  at  this  ticket  a  little  bit,  and  Aaron  will  give  us  what  they  found  preliminary.

Aaron Diefes [46:41]: I  asked.  I  mean,  uh,  Noah  said,  just  said  it,  like,  I'm  just  reading,  um-

Aksana Rahouski [46:46]: Yeah

Aaron Diefes [46:46]: ...  here  that,  like,  he  found  that  on  that  specific  device,  uh,  the  T-Mobile  or  the  T-Mobile  SIM  was  marked  as  inactive,  and  the  Verizon  was  marked  as  active.  Um,  so,  like,  it  was,  it,  it  is  showing  a  Verizon,  like,  signal,  but  that's  because,  like,  that's  what's  active,  I  guess.

Aksana Rahouski [47:06]: 'Cause  it's-

Devon D'Andrea [47:06]: Yeah.  That,  that  is  the  one.  So  this,  this  is,  this  is,  this  contradicts  that  though.  So  here's  one  that  is  a  T-Mobile-

Aksana Rahouski [47:16]: Mm-hmm

Devon D'Andrea [47:16]: ...  device.  We've  got  T-Mobile  SIM  active,  Verizon  SIM  inactive.  It's  coming  in  on  this  IP  space,  which  is  our  T-Mobile  IP  space,  10.20.-

Aksana Rahouski [47:28]: Yep

Devon D'Andrea [47:28]: ...  whatever,  whatever/whatever.

Aksana Rahouski [47:31]: Mm-hmm.

Devon D'Andrea [47:31]: But  you're  showing  it  as,  uh...  Oh,  I'd  have  to  go  back.

Adam Curcie [47:36]: You  gotta  go  back.

Devon D'Andrea [47:38]: Um,  you're  showing  it  as  53%  on  Verizon.

Aksana Rahouski [47:48]: Huh.  Okay.

Devon D'Andrea [47:50]: But  that  check-in  is  on  T-Mobile.

Aksana Rahouski [47:53]: Okay.  Well,  that's  a  good-

Adam Curcie [47:55]: Yeah.  It,  yeah.  Are,  are  you  just...  Oh,  you're  talking  about  the  VZW  or  the  ATT  at  the  end?

Devon D'Andrea [48:01]: Yes.

Adam Curcie [48:01]: Yeah.  Oh,  yeah.  That,  that  makes  sense.  I  don't  think  we  ever  factored  T-Mobile  into  that.  Did  we?

Devon D'Andrea [48:06]: I  don't  think  we  did.

Adam Curcie [48:08]: That's  why-

Aksana Rahouski [48:08]: Yeah.

Adam Curcie [48:09]: That's  why  we  don't,  we  needed  it.

Aksana Rahouski [48:10]: Are  we,  are  we  defaulting  to  VZV,  VZW?

Adam Curcie [48:12]: Well,  I,  but  I,  if,  like,  if-

Devon D'Andrea [48:14]: Yeah.  Unless  it's  like-

Adam Curcie [48:16]: I  think-

Devon D'Andrea [48:16]: ...  the  IP  is  100  dot  something,  it's  gonna  be  AT&T,  otherwise  VZW.  That's  the  only  logic  that  we  had.

Adam Curcie [48:22]: Yeah.  It's  gotta...  I  think  the  logic  that  we  originally  built  this  around  was  10.64  is  AT&T,  everything  else  is  Verizon.  So-

Aksana Rahouski [48:29]: Gotcha.

Adam Curcie [48:30]: Yeah.

Aksana Rahouski [48:30]: So  which  explains  now  when  we  added  T-Mobile,  we  just  didn't  refactor  our  index  page  to  reflect  three.  It's  still-

Adam Curcie [48:37]: Yeah.

Aksana Rahouski [48:37]: The  logic  is-

Adam Curcie [48:38]: Yeah

Aksana Rahouski [48:39]: ...  built  for  two.

Devon D'Andrea [48:40]: So  if  you  are-

Adam Curcie [48:41]: Yeah.  You  guys  can  put  a  very  similar  logic,  you  know,  argument  in  for  10.20.  All  of  our  T-Mobile  traffic  will  be  on  10.20/16

Aksana Rahouski [48:52]: Is  that,  is  that  how  we  are  identifying  them?  So  like  by-

Adam Curcie [48:55]: You  absolutely  can

Devon D'Andrea [48:56]: Yeah.

Aksana Rahouski [48:57]: Okay.

Devon D'Andrea [48:57]: It  is  how  we  are  now.  Yeah.

Aksana Rahouski [48:59]: So  we  need  to-

Adam Curcie [49:00]: Yeah

Aksana Rahouski [49:00]: ...  add  like  in  our  code  somewhere  to,  to  like  switch  for  three  now,  for  two  now  based  on  the  IP.

Adam Curcie [49:06]: Correct.

Aksana Rahouski [49:07]: Okay.

Adam Curcie [49:07]: Exactly.

Aksana Rahouski [49:08]: All  right.  Sounds  good.  Okay.  Well  then-

Adam Curcie [49:11]: Yeah,  and  I,  I,  I  can't  foresee  any  time  in  the  future  where  I  won't  be  able  to  provide  you  with  the  IP  pools  because  everything's  gotta  be  static  and  it's  all  gotta  be  separate,  you  know-

Aksana Rahouski [49:22]: Mm-hmm

Adam Curcie [49:22]: ...  subnets.  So  I  mean,  it'll  be  years  before  we  have  to  look  at  even  thinking-

Aksana Rahouski [49:27]: Okay

Adam Curcie [49:28]: ...  about  retooling  this,  so.

Aksana Rahouski [49:29]: Gotcha.

Devon D'Andrea [49:30]: I,  um,  I  just  wanna  state  that  according,  uh,  you  know,  and  what  Aaron  stated  does  mat-  does  line  up  with  you  guys  when  you  looked  at  it  because-

Aksana Rahouski [49:38]: It  does,  yeah

Devon D'Andrea [49:39]: ...  it,  it  would  appear  just  by  coincidence  that  these-

Aksana Rahouski [49:42]: Got  it

Devon D'Andrea [49:42]: ...  two  devices  in  the  screenshot  have  actually  since  been  converted  to  Verizon  only.

Aksana Rahouski [49:47]: Gotcha.  Yeah,  but-

Adam Curcie [49:49]: Yeah,  but  you  see...  I  was  just  gonna  say-

Devon D'Andrea [49:51]: You  can  see  in  the  screen,  you  can  see  in  the  screenshot  why  it's  not  reporting  right,  'cause  you  have  a  check-in  on  10.20,  but  if  I  pull  these  two  up  right  now,  Adam,  a  number  of  ATM  link  assists  just  the,  just  the  other  day  to  convert-

Adam Curcie [50:03]: Yeah

Devon D'Andrea [50:04]: ...  to,  uh,  T-Mobile  to  Verizon.  So  if,  if  Horus  is  looking  into  these  two  examples  that  John  provided,  they're  gonna  get  confused,  which  I  think  they  already  did.

Adam Curcie [50:14]: Yeah.

Aksana Rahouski [50:15]: Okay.  Okay.  Well,  this  should  not  be  a  difficult  thing  to  fix.

Devon D'Andrea [50:21]: Cool.

Aksana Rahouski [50:24]: Okay.  Look  at  us  on  time.

Devon D'Andrea [50:27]: I  know.  Look  at  that.

Aksana Rahouski [50:27]: Yes.  I-

Adam Curcie [50:29]: But  what,  what's,  what's  I  guess  interesting  is  I  guess  stuff  that's  T-Mobile  only  does  show  a  TMO.  Is  that  what  it  is?

Aksana Rahouski [50:38]: Yeah,  that's  true.

Devon D'Andrea [50:38]: Uh,  how?

Adam Curcie [50:40]: 'Cause  I'm  looking  at...  Like,  if  you  just  type  10.20  into  browse  devices,  there  are  devices  that  report  a  TMO  signal.

Aksana Rahouski [50:48]: Really?

Devon D'Andrea [50:48]: Oh,  I  see.

Adam Curcie [50:49]: But  it-

Devon D'Andrea [50:49]: So  maybe-

Adam Curcie [50:50]: Yeah

Devon D'Andrea [50:50]: ...  we  did  that,  but  we  just  didn't  do  the  IP  thing.

Adam Curcie [50:52]: Yeah.

Aksana Rahouski [50:53]: Oh,  okay.

Adam Curcie [50:54]: I  think  it  just,  I  think  if  it's  T-Mobile  only,  it  does  say  TMO,  but  there's  just  not  any  like  logic  in  the,  uh,  you  know,  the  part  that-

Aksana Rahouski [51:02]: Yeah

Adam Curcie [51:02]: ...  actually  chooses  for  instance  as  a  dual  carrier.

Aksana Rahouski [51:06]: Yeah,  I'm  guessing  we're  just  like  not  handling  dual  correct.  That's  what  it  is.

Adam Curcie [51:13]: But,  but  at  least  the  T-Mobile-

Devon D'Andrea [51:15]: Oh,  yeah,  look  at  that.  You're  right.

Adam Curcie [51:16]: Yeah.  Yeah.

Devon D'Andrea [51:18]: Well,  it's  dual,  but  it's  not  dual.

Adam Curcie [51:21]: It's  dual,  but  it's  not  dual?

Devon D'Andrea [51:23]: It's  the  presence  of  a  Verizon  SIM.

Aksana Rahouski [51:27]: That's  a,  there's  a  d-  double  definition  to  dual.

Adam Curcie [51:31]: I  know.

Aksana Rahouski [51:33]: Yeah.

Devon D'Andrea [51:33]: Yeah.

Adam Curcie [51:33]: We  gotta,  we  gotta  do  a  whole-

Aksana Rahouski [51:35]: Tutorial.

Devon D'Andrea [51:37]: It's  dual,  but  it's  not  dual.  It's  getting  a  dual  config  if  it's  Verizon  only  or  that...  'Cause  it,  it's  all  very  confusing.  And  the  fact  that-  ...  we  labeled  our  SIMs  as  active  or  inactive,  it  confuses  everybody  that  we  work  with.

Adam Curcie [51:50]: Yeah.  We,  we,  we  do  kinda-

Devon D'Andrea [51:52]: Yeah

Adam Curcie [51:52]: ...  have  to  maybe  take  a  look  at  some  point,  not  a  high  priority,  but  like  just  the  way  we  have  everything  kind  of  phrased  and,  or  termed  for  some  of  these  things  does-

Devon D'Andrea [52:03]: This  doesn't  make  sense  to  me

Adam Curcie [52:04]: ...  make  our  own  heads  spin  at  times,  but...

Aksana Rahouski [52:08]: Yeah.  And  the  more-

Devon D'Andrea [52:09]: This  doesn't  make  sense  to  me.

Aksana Rahouski [52:10]: Yeah.  Usually,  like  the  more  multidimensional  it  becomes,  the  more  clear  you  wanna  be  on  your  like  the  word  that  you  use  and  the  definitions-

Devon D'Andrea [52:20]: Right

Aksana Rahouski [52:20]: ...  you  use.

Devon D'Andrea [52:21]: Yeah.  Yeah.  That's  what  it  is.  If  it's,  if  it's  on  T-Mobile,  but  there's  a,  there's  a  Verizon  SIM  present,  it  will  send...  The  slash  signal  strength  will  be,  will  be,  will  show  that  it's  on,  uh,  that  it's  Verizon.  If  it's  on  T-Mobile  and  there's  no,  and  there's  no  Verizon  SIM  present,  AKA  this  one,  it  accurately  shows  that  it's  a  T-Mobile  signal  strength.

Adam Curcie [52:44]: Are  you  sharing?  Why  don't  I  see  your  screen?

Devon D'Andrea [52:46]: Ah,  God.  Ah.  It's  okay.  I  mean,  I-

Adam Curcie [52:50]: You  said  AKA  this  one

Devon D'Andrea [52:52]: ...  we,  we  got  it  I  think.  Sana  can  probably-

Aksana Rahouski [52:54]: Yeah.

Devon D'Andrea [52:55]: ...  see  my  screen  regardless.  Um,  this  one  here  is,  uh,  is  showing  Verizon  less  signal  strength,  and  there  is  a  T-Mobile  active  and  a  Verizon  inactive.  This  one  here,  Herbin  Martin  box,  shows  T-Mobile  on  the  signal  strength  act,  that's  what  we  want,  but  when  you  go  in,  we  did,  we  took  the  Verizon  SIM  out.  It's  probably  down  in  the  admin-

Aksana Rahouski [53:25]: It's  here

Devon D'Andrea [53:25]: ...  notes  somewhere.  Yeah,  it's  over  here.

Adam Curcie [53:26]: Yeah.

Devon D'Andrea [53:26]: That  was  from  our  early  days  of  figuring  out  how  the  hell  to  do  this.  So  we  took  the-

Aksana Rahouski [53:30]: Oh,  got  it

Devon D'Andrea [53:30]: ...  Verizon  SIM  out  completely.  And  if  with  that,  in  that  scenario,  you  get,  you  get  an  accurate,  accurately  stated  T-Mo-

Aksana Rahouski [53:38]: That's  it

Devon D'Andrea [53:39]: ...  on  here.

Aksana Rahouski [53:39]: There  is  no  SIM,  period.  Okay.  Oh,  sounds  good.

Devon D'Andrea [53:44]: So  there  is  kind  of  a  couple  layers  there.

Aksana Rahouski [53:48]: Yeah,  we  can  clean  it  up  so  it's  more  straightforward.

Devon D'Andrea [53:52]: Sweet.

Aksana Rahouski [53:53]: Okay.  Okay.  Sounds  good.  I  think  we're  good  then.  We  have  our  priorities  and  we're  perfectly  aligned  I  feel  like.

Devon D'Andrea [54:00]: All  right.

Aksana Rahouski [54:01]: All  right.  Thank  you  guys  so  much.

Devon D'Andrea [54:03]: Thanks  everybody.

Aksana Rahouski [54:04]: Bye.

Devon D'Andrea [54:04]: Bye.