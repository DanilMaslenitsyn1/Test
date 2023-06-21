#Доп. документ

Задача 1:

Решение: Задача решается путем копирования перка TOUGH_BODY со следующими изменениями:

переименование перка и его единственного триггера в TOUGH_HEAD
константа defenseDamageFactor изменена на -15850 (log от 1/3 по основанию 2)
константа defense изменена на "HeadDefense" (удар в голову)
константа chance изменена на 0.25 (шанс 1/4)
в условии срабатывания триггера TOUGH_HEAD удалено свойство event.Block и добавлено свойство event.Critical (т.к заблокированный удар не может быть критическим, следовательно, свойство блока излишне)

Задача 2:

Решение: Задача была решена путем добавления нового триггера для удара с оружием:

для удобства константа attackBoostChance переименована в unarmedAttackBoostChance
триггер FITNESS_ATTACK переименован в FITNESS_UNARMED_ATTACK
добавлена константа weaponAttackBoostChance = 0.02
добавлен триггер FITNESS_WEAPON_ATTACK, идентичный триггеру FITNESS_UNARMED_ATTACK, за исключением шанса срабатывания (weaponAttackBoostChance) и свойства event.Animation=="Weapon" Таким образом, для данного перка, при срабатывании события "PreHit" проверяются условия для двух триггеров:
Если удар совершен без оружия - с шансом 0.3 срабатывает триггер FITNESS_UNARMED_ATTACK
Если удар совершен с оружием - с шансом 0.02 срабатывает триггер FITNESS_WEAPON_ATTACK

Задача 3:

Решение: Задача была решена следующим путем:

добавлен триггер HELM_BREAKER_INITIAL_RESTRICTION, всегда вызываемый при срабатывании события "FightStart", этот триггер создает иконку баффа "Head_Breaker_restriction" над врагом, которая запрешает срабатывание триггера HELM_BREAKER_INITIAL_BOOST, ответственного за накладывание дебаффа на противника
добавлен триггер HELM_BREAKER_INITIAL_BOOST, который срабатывает с шансом chance = 0.2 если на противнике не присутствуют эффекты данного перка ("Head_Breaker_restriction", "Initial_Boost_Icon", "Icon"), удар приходится в голову и не заблокирован. Этот триггер накладывает на врага эффект "Initial_Boost_Icon", который длится initialBoostFrames = 60 (1 секунду)
добавлен триггер HELM_BREAKER_INITIAL_BOOST_END, который срабатывает при любом незаблокированном ударе в голову противника, если при этом на него был наложен эффект "Initial_Boost_Icon" и при срабатывании увеличивает урон от удара на attackDamageFactor*2
изменена логика поведения триггера HELM_BREAKER_BOOST, теперь он срабатывает при событии "ModExpires", если event.Name="Initial_Boost_Icon" (т.е при истечении 1 секунды после приобретения эффекта) и накладывает старый бафф "Icon", который длится iconFrames - initialBoostFrames (т.е 300 фреймов - 60 фреймов = 240 фреймов, или 4 секунды)
триггер HELM_BREAKER_BOOST_END оставлен без изменений: он срабатывает при любом незаблокированном ударе в голову противника, если при этом на него был наложен эффект "Icon" и при срабатывании увеличивает урон от удара на attackDamageFactor
.

Задача 4:

Решение: судя по коду этот перк должен функционировать следующим образом: При любом ударе, перед подсчетом урона, с шансом 0.3 на владельца перка накладывается новый экземпляр баффа, незначительно увеличивающего урон от ударов. Этих баффов может накладываться неограниченно много (возможно это плохо с точки зрения баланса), пока игроку везет. Как только шанс наложения баффа не срабатывает (random()>=0.3), с персонажа снимаются все "стаки" баффа и его иконка.

В данной реализации присутствует некорректное поведение: сперва проверяется random()<0.3 и при срабатывание создается событие "success", затем сразу же проверяется уже другое значение random()>=0.3 (значение функции random() не зависит от того значения, что эта функция вернула в первом триггере), и если эта функция вернула true, то создается событие "fail", а в результате обработки триггера, подписанного на это событие все баффы удаляются. Простыми словами: в результате одного удара бафф может быть наложен и сразу же снят, чего быть не должно. Самым простым способом решения этой проблемы по моему мнению являются следующие изменения:

В блок action триггера RAGE_CHECK_SUCCESS добавлена функция ModFlag (Name = "Rage_success", Frames=1), т.е. при успешной проверке condition на персонажа вешается флаг "Rage_success" на один кадр
Триггер RAGE_CHECK_FAIL изменяется на RAGE_CHECK, condition триггера меняется на return event.Target=="Enemy" (чтобы он срабатывал каждый удар по противнику), action заменяется на CreateEvent ( "Check" )
Триггер RAGE_FAIL изменяется на RAGE_FAIL_CHECK, event меняется на return "Check", условие меняется на return not ModExists(Name = "Rage_success") (если при ударе перк не сработал, то бафф "Rage_success" на этом кадре не наложился, а значит триггер RAGE_FAIl должен сработать и удалить баффы с персонажа)
Примечания: - в триггере RAGE_SUCCESS_DAMAGE вместо attackDamageFactor (константа, объявленная в перке) указана константа defenseDamageFactor, что вызовет ошибку (вероятна опечатка, не влияющая на логику перка, но блокирующая возможность его работы).

В триггере RAGE_SUCCESS_ICON в условии используется функция ModExists ( "Icon" ) с неверным синтаксисом, должно быть ModExists (Name = "Icon" )

Задача 5:

Решение: перк работает следующим образом: после каждого удара, если целью удара был владелец перка, на сцене отсутствует мод "Shielding" (т.е. его нет на противнике) и функция random() вернула число больше шанса срабатывания chance=0.3, то на противника накладывается дебафф "Shielding", который модифицирует его урон в 2^(-10000/10000), т.е уменьшает в 2 раза на 5 секунд (300/60).

Для того, чтобы шанс срабатывания был пропорционален урону не хватает свойства damage (подсчитанного урона) у события "PostHit". Соответственно нужно передать отделу разработки следующее ТЗ:

Для события "PostHit" добавить свойство "Damage" - значение равное отношению подсчитанного урона от удара к максимальному здоровью игрока, с точностью до четырех знаков после запятой. calculatedDamage/maxHealth в формате "{0:0.####}"

Теперь, если свойство будет добавлено, перк следует изменить следующим образом:

имя константы chance изменить на threshold (порог срабатывания перка)
добавить в condition условие event.Damage> threshold

