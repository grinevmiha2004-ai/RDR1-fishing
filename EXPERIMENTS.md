# Исследование раскручивания перед забросом

## 0.1.46 — отклонена по проверке в игре

Пользователь сообщил, что Джон наполовину проваливается под землю во время раскручивания.
Лог подтверждает повторяющееся playing=1 и продвижение nor_lso_stn_aup_swing.
Это доказывает запуск клипа, но не корректность скелетной позы или реальное изменение координат.

Исходная анимация — gent_nor_lso_stn_aup_swing__SUBTRACT. В 0.1.46 она ошибочно
передана обычному TASK_PLAY_ANIM как полный клип. Для дальнейшего исправления нужно
восстановить базовую позу и штатное добавочное смешивание, а не повторять этот вызов.
Само имя ресурса не доказывает конкретную причину просадки скелета; требуется проверка
пути смешивания. Дополнительно в эксперименте менялся режим наследования поворота
удочки; это тоже следует исключить из следующего изолированного теста.

Не исправлять это принудительным поднятием координат Джона, скоростью или смещением удочки.
Не считать accepted/playing/движение времени доказательством подходящей видимой анимации.
Не выпускать тот же subtractive-клип с произвольным номером слоя без проверки ABI.

Основная подтверждённая версия — 0.1.45, baseline-v0.1.45. Архивы сохранены без изменений.
Логика удержания/отпускания может быть сохранена отдельно от неверного способа проигрывания.


## Дополнительная проверка после теста 0.1.47

Пользователь отклонил кандидаты 1–9. У 0 есть круговое движение, но поза
искажена; пользователь не подтвердил его как нужный клип. Не считать его
выбранным по одному признаку кругового движения.

Просмотрены 1345 декомпрессированных WAS из animationres.rpf. Среди найденных
именованных источников раскручивания лассо обнаружены пеший
`gent_nor_lso_stn_aup_swing__SUBTRACT` и полный верховой
`gent_hrs_lso_stn_aup_swing`. Последний не тестировался и может иметь
неподходящую позу ног. Поиск строк/алиасов не доказывает отсутствие другого
клипа под несвязанным именем. `nor_lso_for_run_swing` из WAT не найден
как самостоятельный ключ клипа во всех проверенных WAS.

Существенное отличие подтверждено не только суффиксом SUBTRACT: у standing
swing AnimationTrack 0x9404C / 0x94428 поле файла +0x1C равно 1, у
подтверждённого броска 0x92B48 равно 0. Сериализатор AnimationTrack
0x140D57D40 и reader 0x140D3C370 сопоставляют это с m_BlendMode
в runtime +0x28. Обработчик 0x140D42330 читает этот режим и при 1
идёт к 0x140CEDF50, в отличие от обычного проигрывания. Эта функция
использует отдельные ресурсы добавочных слоёв компонента +0xB40;
обычный TASK_PLAY_ANIM идёт через 0x140CDE6F0 и primary custom clip +0x9F0.
Следующий содержательный эксперимент — штатное добавочное проигрывание
с проверенными ABI, базовой позой и очисткой. Оно ещё не реализовано;
не подменять его произвольным слоем TASK_PLAY_ANIM или исправлением высоты.
Основная подтверждённая версия остаётся 0.1.45.

## 0.1.52 — корректировка хвата после 0.1.51

Пользователь одобрил результат 0.1.51 и попросил опустить рукоять в кисть и немного наклонить вперёд. Новая калибровка: -12 см вдоль фактической оси joint5, 2,5 см к ладони в сторону переда персонажа; Ready -20° вместо -12°, центр конуса раскручивания -8°, радиус прежний 7,5°. Смещение переводится из мира в текущий базис wrist_r; задаётся штатным SET_ATTACHMENT_LOCAL_OFFSET без пересоздания крепления. Камера, отключённые коллизии, анимации и фаза вываживания .5238 сохранены. Проверки и пересборка исходного архива пройдены; визуальное положение в игре ещё не подтверждено. Основной подтверждённый baseline 0.1.45 сохранён.

## 0.1.53 — боковая поправка и единый наклон

По просьбе пользователя рукоять смещена на 2 см вправо на предоставленном фронтальном снимке (влево относительно Джона), направление следует повороту персонажа. Продольное опускание на 12 см сохранено во всех фазах. Дополнительные -8° наклона теперь применяются ко всем фазовым значениям pitch, включая заброс и вываживание; центр конуса раскручивания остаётся -8°. Коллизии, камера и клипы прежние. Тесты, ABI и идентичная пересборка из исходного ZIP пройдены; визуальная посадка в игре не проверена.

## 0.1.54 — исправлен наклон раскручивания

В 0.1.53 центр конуса оставался -8° при радиусе7.5°, допуская почти вертикальную удочку. Пользователь указал на отсутствие требуемого наклона при раскручивании. Центр теперь -20°, как Ready; pitch во всём цикле -27.5..-12.5°. Остальные фазы, смещение вправо2см, продольное опускание12см, камера и клипы сохранены. Циклический тест и идентичная пересборка исходного ZIP пройдены. В игре не проверено.

## 0.1.55 — больший наклон при раскручивании

По просьбе пользователя центр конуса раскручивания наклонён на40° вместо20°, радиус7.5° сохранён. Диапазон pitch во всём цикле -47.5..-32.5°. Остальные этапы, хват и камера без изменений. Тест полного цикла и идентичная пересборка из исходного архива пройдены; визуально в игре не проверено.

## Подтверждение основной версии 0.1.55

Пользователь: «Отлично, просто превосходно. Теперь берем эту версию за основную. Обнови память». Новая база — 0.1.55, коммит6673fec, постоянный тег baseline-v0.1.55. Архивы неизменны. Предыдущие базы и теги сохранены.

## 0.1.56 — полный конус вместо покачивания

Вершинка обходит вертикальную ось полным кругом: постоянный наклон40°, yaw меняется на полный оборот. Прежние40° вперёд с покачиванием7.5° заменены; плавный вход/выход смешивает направления древка. Хват, продольное опускание12см, другие фазы и камера сохранены. Тест геометрии полного круга и воспроизводимая пересборка пройдены; в игре не проверялось. Пользователь разрешил публикацию на GitHub для прямого скачивания. Основная подтверждённая версия остаётся0.1.55.

## 0.1.57 — обратное вращение и горизонтальная плоскость снасти

По обратной связи пользователя направление конуса удочки развёрнуто. Фаза совмещена с радиальным направлением снасти. Вместо плоскости, перпендикулярной наклонённой удочке, применяется горизонтальная плоскость; поплавок, крючок и вершинка имеют общую высоту после плавного входа, провисание лески при вращении отключено. Переход в полёт, хват и камера сохранены. Проверки полного цикла, направления радиусов, горизонтальности и идентичная пересборка исходников пройдены. В игре не проверено. База0.1.55 сохранена.

## Возврат к 0.1.55 и 0.1.58 — хват для трёх поз

Пользователь предпочёл вернуться к основной0.1.55, экспериментальный конус0.1.56–57 пока отложен. Новая0.1.58 сделана строго от0.1.55. По трём снимкам добавлены отдельные поправки к прежнему хвату в координатах персонажа (вправо/вверх, метры): Ready(-.01,-.005), Wait/Bite(.05,-.01), Fight(-.02,-.015). Переход smoothstep0.65с, при смене цели продолжается от текущего положения, пауза не двигает смещение. Перед забросом/другими фазами поправка возвращается к нейтральной. Сохранены12см опускания, все углы, клипы, фаза.5238, камера и коллизии0.1.55. Проверки переходов, ABI и воспроизводимая пересборка пройдены. Калибровка — оценка по снимкам; точное положение в глубину и визуальная посадка в игре не проверены. Основная подтверждённая база остаётся0.1.55.

## 0.1.59 — уточнение хвата по новым снимкам

Пользователь просит обычную стойку немного ниже, ожидание и вываживание правее на фронтальном виде. Относительно0.1.58: Ready дополнительно2.5см вниз, Wait/Bite и Fight дополнительно3см влево относительно Джона (вправо на предоставленных фронтальных снимках). Итоговые добавочные right/up: Ready(-.01,-.03), Wait(.02,-.01), Fight(-.05,-.015)м. Углы, анимации и все остальные механизмы без изменений. Тесты и идентичная пересборка исходников пройдены, визуально в игре не проверено. Основная подтверждённая база0.1.55 сохранена.

## 2026-11-13 — подтверждённая основная версия0.1.59

Пользователь: «На сегодня просто идеально… сохрани эту версию как основную рабочую, обнови память». Новая основная база0.1.59, коммит3923ed1, постоянный тег baseline-v0.1.59. Сохранить настройки хвата Ready/Wait/Fight и поведение0.1.55; экспериментальный конус0.1.56–57 не возвращать без запроса. Архивы неизменны, предыдущие базы сохранены. Дальнейшие изменения обсудить в следующей сессии.

## 0.1.60 — эксперимент размера рыбы в памяти

Пользователь выбрал исследование runtime-масштаба вместо дополнительных файлов моделей. Найдены prop wrapper vtable1C738E0, getter21A6B0, setter21A6A0 ->234BB0 ->89D6B0, копирующий полную матрицу без нормализации. Тестовая сборка от0.1.59: верхние1/2/3 выбирают0.5/1/2x на той же рыбе. Проверяется точный профиль EXE, отпечатки функций, vtable и поколение handle. Масштаб задаётся через движковый setter после обычного позиционирования/вращения, без общих моделей и патчей кода. Оси нормализуются перед умножением, коррекция рта учитывает масштаб. Проверяется обратное чтение; отказ возвращает штатное положение и вращение, причина логируется. applied=1 означает только прочитанную матрицу, не видимый результат. Тесты матриц и ABI пройдены, архив пересобран идентично. Игра в облаке не запускалась. Основная подтверждённая база0.1.59 сохранена, тег не менялся.

## 0.1.60 подтверждена; 0.1.61 — вертикальная пойманная рыба

Пользователь подтвердил масштабирование0.1.60: «Супер! Оно работает!». Следующая просьба — пойманная рыба должна висеть вертикально. В0.1.61 только Caught за0.35с плавно переходит к Euler(0,0,-90), модельная ось головы-X направлена вверх. Простое добавление roll при произвольном yaw не гарантирует вертикаль, что проверено формулой16FE80. Поворот корректирует существующий world-space mouth offset, масштаб применяется после. Вода/вываживание сохраняют прежнюю ориентацию. Верхние1/2/3 и рабочий setter масштаба сохранены. Сборка, проверки и идентичная пересборка исходного архива пройдены; новая ориентация в игре не проверялась. Основная база0.1.59 сохранена.

## 0.1.61 подтверждена; 0.1.62 — категории веса и автоматический размер

Пользователь подтвердил вертикальную пойманную рыбу0.1.61, затем согласовал50% мелкая<1кг,30% обычная1..5кг включительно,20% крупная>5..10кг. В0.1.62 категория выбирается один раз при забросе отдельным random, затем вес в сотых кг:40..99,100..500,501..1000. Минимум400г сохранён. Масштаб автоматически0.5/1/2, тестовые1/2/3 и TEST FISH удалены. Сохранены подтверждённые setter масштаба и вертикальная позаCaught. Границы и100000 seeded samples проверены; исходный архив пересобирается идентично. В игре новое распределение не проверялось. Основная база0.1.59 сохранена.

## Основная подтверждённая версия0.1.62

Пользователь: «Замечательно! Просто отлично! Теперь это наша основная версия, запомни ее и обнови память». База0.1.62, коммит586903e, тег baseline-v0.1.62. Подтверждены автоматические категории50/30/20%, веса0.40..0.99/1..5/5.01..10кг, масштабы0.5/1/2 и вертикальная пойманная рыба, прежний хват и анимации сохранены. Архивы не перезаписывать; будущие эксперименты отдельно. Предыдущие основные теги сохранены.

## Reel audio: local RPF audio table collector v0.1

User requires playing existing game sound0x01B4B46C from sfx.awc; embedded WAV alternative is not authorized. MagicRDR export fails with Zstandard unknown frame descriptor. Prepared separate Windows x64 RDR-AudioTables-Extractor-v0.1.zip: opens selected3GB archive read-only, reads/decrypts RPF6 TOC, selects config sounds/game tables by JOAAT basename hashes, collects stored bytes and flags/offsets into small ZIP. No decompression needed on user PC, no automatic upload, no game-file modifications. Tested parser on1352-entry reference RPF,64-bit offsets beyond2GB and independent ZIP extraction/CRC. Windows binary compiled but not run here; actual user audio RPF not available. Await generated RDR_AudioTables_*.zip to resolve native audio event. Draft mod0.1.63 with embedded WAV remains local/unpublished and must not be distributed as solution to native audio request. Main confirmed baseline0.1.62 unchanged.

## 0.1.63 — катушка через точное игровое событие

Получен ZIP локального экстрактора от пользователя. Успешно декодирован raw Deflate(-15): причина Unknown frame descriptor — MagicRDR выбирал Zstandard для PC-таблицDeflate. В sounds.dat20 найдено точное соответствие0x01B4B46C: HUD_MENU_MENU_060308_DPAD_03 (offset402465,length27,type12,bankSTATIC\\SFX). Второй алиасHUD_HONOR_HUD_MENU_MENU_060308_DPAD_03. Выпущенная0.1.63 использует только игровые нативы: GET_SOUND_ID, PLAY_SOUND_FRONTEND_PERSISTENT(id,name), HAS_SOUND_FINISHED, STOP_SOUND и RELEASE_SOUND_ID. Встроенного WAV, WinMM и дополнительных аудиофайлов нет; прежний локальный WAV-эксперимент удалён из выпуска. При Fight+Reel&&!Ease повтор>=80мс после завершения щелчка; одновременно максимум один принадлежащий моду soundID. Отпускание, Q, выход изFight, F6, пауза и потеря фокуса останавливают/освобождают только этот звук. Подтверждённые функции0.1.62 сохранены. Тесты жизненного цикла, точная привязка сэмпла и ABI, сборка и идентичная пересборка исходников пройдены; слышимость в игре ещё не проверена. Основная подтверждённая версия0.1.62 сохранена.

## 0.1.64 — случайное чередование выбранных щелчков

Пользователь отметил подходящими0x062C7D5B,0x0FEF10E1,0x0D414B88,0x0B2CA65B,0x04E7D9D6,0x0F88EF19;0x1224B450 не брать. Реализовано случайное чередование ровно этих шести через игровые события DPAD01/02/04 и NAV01/03/04. Независимый xorshift, без одинаковых подряд, не влияет на Game RNG/вес рыбы. Проверены соответствия всех шести по пользовательским таблицам,60000 выборов, жизненный цикл soundID и идентичная пересборка. Остальное от0.1.63 без изменений. В игре звучание новой сборки не проверено; основная подтверждённая0.1.62 сохранена.

## 0.1.65 — замедление катушки

По обратной связи пользователя увеличен минимальный интервал щелчков с80 до160мс. Случайные шесть событий и жизненный цикл звука прежние. Тест частоты обновлён и пройден, исходный архив пересобран идентично. Новая частота в игре не проверялась. Основная0.1.62 сохранена.

## 0.1.66 — проверка веса/размера и явная нормализация1x

Пользователь сообщил о визуально огромной рыбе1.37кг. В коде1.37кг соответствуетnormal1x; диапазоны/вероятности50/30/20% проверены. Найден пропуск явного масштабного setter для1x (условиеfishScale!=1), убран. Теперь0.5/1/2x задаются одинаково явно после rotation, включая нормализацию входных осей. Лог дополненobject,weight,category,requested,observed,applied. Без runtimeлога точная причина визуального случая не установлена; не выдавать потенциальное исправление за доказанный корень проблемы. Тест2->1->.5->2->1, границы1/5кг, распределение и ABI пройдены; пересборка архива идентична. Катушка160мс0.1.65 сохранена. Основная подтверждённая0.1.62 неизменна.

## 0.1.67 — свободная камера и выбор цели во время удержания E

Пользователь подтвердил звуки, попросил свободную камеру при раскручивании и выбор направления. При bind спина CastControls больше не захватывается; управление остаётся как было. Нажатие E начинает зарядку без фиксации воды/поворота; камера читается во время удержания. На одноразовом releaseEdge prepareCast повторно читает текущую камеру, проверяет воду и фиксирует цель. После отпускания штатный поворот/бросок с прежним владением управлением. Если воды/направления нет — отмена вReady с освобождением только своего состояния, без заброса в старую цель. Сохранены катушка160мс и явный масштаб1x0.1.66, веса/вероятности и позы. Проверки длинного удержания/единственногоrelease/quicktap/reset, SDK и идентичная пересборка пройдены. Реальная камера и анимация требуют проверки в игре. Основная подтверждённая база0.1.62 сохранена.

## 0.1.68 — удалён DPAD02

Пользователь попросил убрать0x0FEF10E1. Удалён HUD_MENU_MENU_060308_DPAD_02 из катушки, остаются пять звуков со случайным выбором без одинаковых подряд.160мс и свободная камера0.1.67 сохранены. Тест распределения по пяти, соответствия события/сэмпла и идентичная пересборка пройдены. Основная0.1.62 сохранена.

## 0.1.69 — удалён NAV01

По просьбе пользователя удалён0x0B2CA65B. Остались четыре:0x062C7D5B,0x0D414B88,0x04E7D9D6,0x0F88EF19.160мс, отсутствие повторов подряд, свободная камера и остальные настройки сохранены. Тесты, соответствия событий и идентичная пересборка пройдены. Основная0.1.62 сохранена.

## 0.1.70 — ручной тест пяти звуков

Пользователь слышит лишний звук и просит фиксированный выбор по клавишам плюс0x04766715. Тестовая0.1.70: верхние1=0x062C7D5B,2=0x0D414B88,3=0x04E7D9D6,4=0x0F88EF19,5=0x04766715(FOLEY_DOORS_SAFE_TUMBLER_TURN_04). При включённой рыбалке нажатие выбирает и однократно проигрывает звук; при R вFight повторяется только выбранный, случайный выбор отключён. HUD и лог показывают номер/хеш. При выборе новый звук заменяет предыдущий принадлежащий моду голос. Сохранены160мс, остановка наkey-up/Q/паузе/F6, свободная камера и остальная механика. Тесты выбора/жизненного цикла, точные пять соответствийсэмплам, SDK и идентичная пересборка пройдены. В игре тестовая сборка не проверялась. Основная0.1.62 сохранена.

## 0.1.71 — повтор на сохранённом soundID без внешнего STOP между щелчками

Пользователь слышит перепады громкости даже при повторе одного звука. Старый код между каждым запуском вызывалSTOP/RELEASE/allocate и принудительно прерывал незавершённые события через0.5с. В0.1.71 один soundID сохраняется на время удержанияR, повтор только послеHAS_SOUND_FINISHED и>=160мс. Убраны внешниеSTOP/RELEASE/allocate между повторениями и таймаутное обрезание. Естественное окончаниеpreview освобождается безSTOP. Key-up/Q/F6/пауза/фокус/конецFight сохраняютstop+release. Ручные1–5 из0.1.70 сохранены. Тест1000update подтверждает одну allocation и отсутствие внешнихSTOP/RELEASE между щелчками, незавершённый звук не обрезается. Это исправление жизненного цикла, не доказательство причины слышимого перепада. В игре не проверено. Глобальная громкость и общая аудиобаза не меняются. Основная0.1.62 сохранена.

## 0.1.72 — возвращён случайный выбор, добавлен звук броска

Пользователь сообщил, что0.1.71 звучит намного лучше и проблема вроде решилась. Просит вернуть случайный порядок с повторами подряд и звук ветра при броске лассо (при раскручивании уже есть). Катушка случайно выбирает из пяти текущих событий, собственныйPRNG, повторы разрешены,160мс и сохранённыйID/естественное окончание0.1.71 сохранены. Тестовые1–5/HUD убраны. В пользовательской таблице найден LASSO_ACTIONS_THROW_MASTER ->STATIC_WEAPONS_LASSO_THROW_01, sample0x040BA803. Отдельный принадлежащий моду одноразовый звук запускается при фактическомcastFlightReleased вCast, playedlatch сбрасывается на новом удержанииE; при отмене наведения не играет. Естественное завершение освобождается безSTOP, пауза/F6 очищают свой канал. Проверки выборки с повторами, стабильного канала, одноразового броска, точной связи события и идентичная пересборка пройдены. Слышимость/синхронизация новогоwhoosh требуют проверки в игре. Основная0.1.62 сохранена.

## 0.1.73 — вода по ситуациям и каналы катушки по клипам

Лог пользователя0.1.72 проверен: последние5уловов0.43/.5x,8.18/2x,4.16/1x,3.22/1x,4.03/1x,50масштабных записей без отклонений. Рыбный код/размеры не меняются. Звук броска подтверждён пользователем и сохранён.

По запросу воды: приземление снасти в воду Cast->Wait с released выбирает ровно один из0x008CE18D/0x00E2AE50/0x014E76D5. Начало активного рывкаFight (общая с Game функцияfishPulling) выбирает0x016EF8B8/0x00E2AE50, не каждый кадр; поимкаFight->Caught выбирает0x0F8C00C2/0x016EF8B8. Всеalias/sample соответствия проверены по пользовательской dat20; отдельныйPRNG не влияет на рыбу. Голоса воды принадлежат моду, естественное завершение освобождается безSTOP, пауза/F6 очищают.

Перепады громкости вернулись после случайной смены событий на одном ID. В0.1.73 каждый из пяти клипов имеет собственный постоянныйID, не получает другое событие; повтор только после естественного завершения,160мс, случайные повторы разрешены. Макс5 выделенныхID, одновременно одна повторяющаясякатушка;stop/release только при отмене, диагностика хеша/ID в логе. Это целевое изменение жизненного цикла; реальная причина/нормализация амплитуды не доказаны и требуют проверки на слух. Глобальная аудиобаза и громкость не трогаются. Проверки триггеров/наборов/каналов/соответствий/SDK и идентичная пересборка пройдены. Основная0.1.62 сохранена.

## 0.1.74 — приоритет: совместное звучание воды и катушки

Пользователь сообщает: всплеск рывка прекращает катушку, приоритет — устранить конфликт. Дополнительно просит008CE18D тише и00E2AE50 из приводнения перенести вCaught, оставивStruggle. Вода переведена с FRONTEND на POSITIONAL owned playback, катушка остаётся frontend на своих пятиID. Используется точный путь публичного5B05E3E0 через guardedDEB2B0 с112-byte params, чтобы дополнительно задать толькоgain=-6dB для008CE18D. Код/EXE проверяются, позиция16-bytealigned, остальные параметры совпадаютnativeE33D40. Никакого вмешательства в глобальныймикшер, общие параметры или WAV. При неподдержанномпрофиле пропуск воды сreason влоге, без возврата кстаромуfrontend.

Итоговые группы: Landing008CE18D/014E76D5; Struggle016EF8B8/00E2AE50; Caught0F8C00C2/016EF8B8/00E2AE50. Звук броска/рыба/камера/катушка160мс сохранены. Проверка совместного владенияID на общемmockpool подтверждает, что активнаявода и её остановка не прекращают катушку; проверены точные ABI/константы/gain/табличные связи. Пересборка исходногоZIP идентична. В игре фактическое смешивание не проверено — не считать mock доказательством отсутствия enginevoice-limits. Основная0.1.62 сохранена.

## 0.1.75 — фиксированный тест и A/B способа катушки

Пользователь сообщает, что0.1.74 не устранила затихание/подавление катушки при рывке; просит опять фиксированный выбор звуков и исследование конфликта. Подготовлена отдельная тестовая0.1.75. Клавиши1–5 фиксируют те же пять и дают preview; приR вFight повторяется только выбранный.6toggleAutoStruggle (выключение стопит только голос рывка),7 вручную воспроизводитStruggle,8переключаетMENU/CHANNEL. MENUпоумолчанию использует стандартный публичный2E458F74one-shot раз>=160мс, без ID/ожиданияHAS_SOUND_FINISHED; CHANNEL оставляет прежний retainedID. ВMENUуже запущенный краткийone-shotдоходит до естественного конца при отпускании, новые не запускаются. ТестоваяHUD показывает выбор/режим/воду. Лог каждые0.5сFight фиксируетR/Q/wanted/ID/finished/struggleID. Это различает прекращение реального вываживания приQ, зависшее ожиданиеID и слышимое подавление микшером. Не считать тест доказанным исправлением. Вода/группы/-6dB/бросок/рыба/камера сохранены от74. Проверки выбора, расписанияMENU, SDK/ABI и идентичная пересборка пройдены. Основная0.1.62 сохранена.

## 0.1.76 — явный gain MENU и усиленные рывки

Пользователь подтвердил, чтоMENUне отключается всплеском,CHANNELотключается; на одном клипе бывают серии громко/тихо. Найдено значимое различие вEXE: обычный PLAY_SOUND_FRONTEND/E33810 вычисляет gain=(contextScalar-1)*45; INITPARAMS/E33910 берёт явныйfloat0 из входной структуры. Катушка переведена на49053A94(event,28byteparams), gain0dB, остальные поля нули; без записи глобальной громкости/микшера/OS. Обычные master-регуляторы downstreamсохраняются. Это подтверждённое изменение параметров запуска, не доказательство устранения всех слышимых перепадов. MENU/RNGпятисэмплов/повторы/160мс сохранены; тестовые1–8/CHANNEL убраны.\n
Вторая просьба — сильнее рыба/удочка при вырывании. FightMotionсинхроненfishPulling: вход.18с/выход.28с, rodPitchдо8°,rollдо6°, общийконецснасти sideдо14см,yдо8см, defaultfishflutterдо28°. Поплавок/крючок/рыба двигаются вместе, леска остаётся соединённой; рыбийyaw/вес/размеры/хват/коллизии/камера/Caughtverticalне меняются. Дляcustommouthoffsetсохраняется запретflutter, но endpointдвижение работает. Пользователю уточнено: фиксируется только относительный gain щелчка, не общая громкость игры/Windows/колонок. Проверки ABI/структур/динамическогоvsявногоgain, ограниченийдвижения и идентичная пересборка пройдены. В игре новаясборка не запускалась. Основная0.1.62 сохранена.

## 0.1.77 — RDR1 water splash preview

Built from0.1.76. Ten confirmed names from user's vfx.rpf (SHA256
40b2616bf50396d1706b005af82ffd3d1606a95cb2fdaf8f4f7e0073351e9932),
vfx/rdr2_core.fxlist: ped_water_thrash, ped_water_wade, ped_water_entry,
bullet_water, amb_splash, amb_water_splash, amb_water_rocksplash,
amb_water_rocksplash_2, anim_water_toss, wagon_water_wade.
Keys1–0 choose/preview after a cast; T replay, Y toggle automatic at pull onset,
Backspace stop/disable automatic. HUD labels choice; log records native START
result and surface coordinates. Manual preview capped3s, auto1.5s, one owned
emitter; manual cooldown protects preview from auto overwrite. Cleanup only
owned emitter/layout, no global FX reset. Native packed vector ABI checked in
user EXE; START vector is Euler rotation, CREATE vector is position.
Portable scheduler tests, all ten catalog names, SDK/native exports/imports,
source rebuild byte comparison, ZIP CRC passed. No in-game validation here.
Main confirmed baseline0.1.62 remains unchanged. Incomplete FX collector draft
was unnecessary once user uploaded vfx.rpf; not distributed.

## 0.1.78 — selected fish-size splash effects

User confirmed v0.1.77 previews: key1 for normal fish, key4 for small fish,
key8 for large fish. Bound to game.weight using exact model category thresholds:
<1kg bullet_water; 1–5kg inclusive ped_water_thrash; >5kg amb_water_rocksplash_2.
Automatic burst at struggle onset, same scale1/lifetime1.5s as0.1.77 automatic.
Removed numeric/T/Y/Backspace preview inputs and preview HUD. Immediately stop
on leaving Fight; owned emitter/layout cleanup preserved. Audio and fish motion
unchanged. Tests cover weight boundaries, repeated pulling, rest/restart and
zero-delta pause. SDK/catalog validation, ZIP CRC, identical source rebuild pass.
Game execution is unavailable in cloud; integrated mapping awaits user check.
Confirmed main baseline0.1.62 retained; v0.1.77 test archive retained for rollback.


## Подтверждённая основная версия0.1.78

Пользователь прямо подтвердил0.1.78 как новую основную рабочую версию.
Коммит сборки caf7135, постоянный тег baseline-v0.1.78. Архивы неизменны.
Дальнейшие эксперименты выпускать отдельно, не перезаписывать базу/тег.
WORKING_BASELINE.md обновлён, предыдущие базы и теги сохранены.

## 0.1.79 — underhand molotov preview, based on0.1.78

Removed008CE18D from Landing, leaving014E76D5. Metadata of removed sample retained.
Preview: F6 then1 standing underhand;2–9 eight directional moving underhand
variants;0 normal standing comparison; Backspace cancel. Ready/Caught/Lost only,
mini-game frozen during preview; camera remains free. Existing fishing casts
unchanged. Positive rate1, forced loopfalse, timer-1, no seeking/frame freeze;
ends after observed native playback completion, with30s runaway safeguard.
Runtime keys individually verified via actual dyn.was hash records and source
names. Unsuffixed run WAT track is a directional container, not a playable key;
use suffixed actual runtime entries. dyn.was contains MOLOTOV_THROW_DYNAMIC.
Tool verify_throw_preview.py documents all10 hash->source mappings and assetSHA.
Portable task argument and Landing exclusion tests pass, SDK/nativeABI checks
pass; source ZIP rebuild byte-identical, both ZIP CRC verified. RDR unavailable
in cloud; visual suitability awaits user. Baseline0.1.78 untouched.

## 0.1.80 — keep/release catch, underhand throw and impact sound

User confirmed0.1.79 key1. Use dyn/nor_dyn_stn_for_mid_und_trw, nonloop/rate1.
Caught waits indefinitely for Action(E) keep or Ease(Q) release. Total catches
still includes released fish, separate Progress/Kept and Released decision counts.
Keep records mod statistics; game inventory item integration is not implemented.
Release temporarily holds rod at wrist_l, transfers fish from hook to right wrist,
then detaches at initial normalized phase0.48 and flies a1.2m-lift parabolic path
to the existing verified water surface. Preserve scale; remove fish at impact.
Release cue always008CE18D with existing -6dB gain, isolated fourth owned water
sound slot; Landing remains only014E76D5. Size-specific splash at impact retains
its1.5s tail after returning to Ready. No early fake sound on button press.
Release sync phase and left-hand rod placement await in-game visual validation.
Failure before detach preserves caught fish and choice. Cancel/player loss clean
owned sessions/props only. Preview keys removed; main baseline0.1.78 untouched.
Tests cover indefinite choice, cue mapping, phase gate, pause, one impact/exact
flight endpoints, native task args. Catalog/nativeSDK/audio mapping validations,
ZIP CRC and byte-identical source rebuild passed; game unavailable in cloud.

## 0.1.81 — fish release missing-startup recovery

User0.1.80 log: seven requests, four impacts, three Hand-stage6s failures. Old
log has no playing/time samples, so exact failure cause (missing versus stalled
playback) is not proven; generic cast progress fields were not updated on release.
Add Bound frame between animset bind and task request, then at most two startup
retries only while no native playing has ever been observed. Scoped ownership
checked before retry; no replay of an observed clip. Three missing requests fail
boundedly preserving catch. Require0.25s query gap before natural-end decision.
Add stage/playing/phase/age/seen/attempt logs every0.2s. No phase/flight/audio changes.
Tests cover absent start/recovery, pause, no replay after match, finite retry cap.
SDK compatibility, ZIP CRC, identical source rebuild passed. Runtime fix awaits
user validation; baseline0.1.78 unchanged.

## 0.1.82 — standing item draw/stow preview

User confirmed0.1.81 release fix works, then requested lasso/item draw/stow clips
while standing. Twenty mapped runtime hash/source pairs from animationres.rpf:
page1 keys1–4 lasso draw/post/stowpre/stow,5–6 bottle/dynamite draw/stow,
7–8 throwing knife draw/stow,9–0 binocular takeout/putaway. F7 switches page;
page2 keys1–4 pistol draw/post/stowpre/stow,5–7 rifle draw/post/stowpre,
8–0 full inventory use actions bottle/medicine/bait (not pure draw/stow).
Only standing sources, no mounted/run clips. No weapon/item props spawned or
inventory modified by preview. Digits play/replay, Backspace cancel, F6 cleanup.
Ready/Caught/Lost only, not during release; minigame suspended, camera free.
Scoped bind, one-frame Bound barrier, missing-only bounded startup recovery;
natural rate1/nonloop/timer-1, no seek/freeze. Per-set unique streaming request.
Portable page/index/resource ownership mapping tests and SDK validation pass;
all20 asset hash->source mappings verified by verify_item_preview.py. ZIP CRC,
byte-identical source rebuild pass. Game visual validation remains with user.
Fishing/release/audio unchanged. Confirmed main baseline0.1.78 remains intact.

## 0.1.83 — preserved chosen stow, additional bag-reach candidates

User chose0.1.82 page1/key6 as future stow: dyn_base/nor_dyn_stn_hls. Durable
CONFIRMED_ITEM_ANIMATIONS.md records exact key/set/source; page1 unchanged.
Not integrated into normal rod removal yet. User rejected all previous draws
as too short, wanted real reaching into bag. Confirmed lasso draw/post are
~0.333s resource segments. No artificial slowdown or claim they are new clips.
Replaced page2 with bandana pre/post, player give_money, standing lantern draw
(NPC Gped source explicitly labelled), lasso left/right/front-left/front-right
storage preparation, long holster post shared pistol source, dynamite preparation.
These are candidates, not all literal takeout animations. Desired draw remains
unconfirmed. Verified all20 real runtime hash/source pairs, including alias and
mission source exceptions; tests/SDK validation/ZIPCRC/reproducible rebuild pass.
Main baseline0.1.78 unchanged. No game runtime visual validation in cloud.

## 0.1.84 — integrated catch hands and user-specified item sequences

Use numbers from0.1.83 (NOT0.1.82): small keep page1/3, normal1/4, large1/6;
recover after release page2/0; equip page2/0 then2/4; stow page1/9.
Exact mappings recorded in CONFIRMED_ITEM_ANIMATIONS.md and source ItemSequence.
On Fight->Caught, rod attachment moves left, fish anchors to right wrist using
confirmed vertical fishHang transform. Keep .65 phase removes fish once, records
kept count, restores pole right; catchResolved prevents respawn. Release retains
.48 underhand detach/flight/impact008CE18D; afterwards Recover plays then transfers
rod right at.65. Equip creates no tackle props until second animation.45; stow
removes props at.65 then cleans session after native clip completion. Phase values
are initial synchronization choices, not user-validated bag contact frames.
Generic scoped sequence controller uses known Bound frame and missing-only startup
recovery. No numeric/F7/Backspace previews. F6 while active sequence/release is
immediate cancel; otherwise normal animation stow. Camera remains free. Choice
waits for user; pre-store failure preserves catch, committed store never duplicates.
Portable tests cover exact clip keys, weight1/5 boundaries, equip ordering, event
single-fire/reset; SDK and20 original asset mappings verified. ZIP CRC and identical
source rebuild pass. No in-game cloud validation; baseline0.1.78 retained unchanged.

## 0.1.85 — earlier hand/contact events and overlapping rod draw

User says0.1.84 works; requests no long gap before draw step2, earlier pole
removal at bag contact, earlier pole transfer so returning hand already holds it.
Preload all sequence sets at begin. Switch Equip step0->1 at phase0.80 while
still playing if step2 is loaded, bypass natural-end wait0.25s; preserve short
Bound/request frame separation for reliable startup. Natural-end fallback if
streaming is not ready. Stow disappearance0.65->0.30; Keep fish removal+rod-right
and Recover rod-right0.65->0.45. Equip appearance0.45 of second clip unchanged.
No changes to weight thresholds, clip selection, release0.48 or impact sound.
Tests cover Equip-only step0 overlap and earlier single-fire thresholds; SDK,
ZIP CRC and byte-identical source rebuild pass. Actual bag-contact alignment
requires user runtime check; confirmed baseline0.1.78 remains unchanged.

## 0.1.86 — single draw animation and recast candidate preview

User removes first preparatory draw clip: Equip now only index13 ltn_base/
nor_ltn_stn_drw, pole appears at0.45 of that sole clip. No overlap/preparation.
Ten per-key lasso pull-in/tug/throw outcome clips, verified in supplied WAS by
verify_recast_preview.py.1 standing pulin,2 failed standing throw,3 high failed
throw,4 successful throw outcome,5 short standing tug,6 long standing tug,
7 standing pull,8 forward tug walk,9 backward tug walk,0 full grounded-target pull.
Numeric preview in Ready/Wait/Bite/Caught/Lost, no active Equip/Stow/Keep/Recover,
release or Fight; gameplay clock paused during test. Backspace cancel, F6 cleanup.
No cyclic playback/frame freeze; bounded missing-only startup retry and30s watchdog.
Digit0 is the full main fight source, without .5238 hold. Test previews movements
only; actual recast mechanic intentionally awaits user's clip selection.
Weight/keep/release/stow and0.1.85 earlier contact timings retained. SDK validation,
10 real clip source mappings, portable tests, ZIPCRC/reproducible rebuild passed.
No cloud game runtime. Main confirmed baseline0.1.78 remains unchanged.

## 0.1.87 — retrieve4 and automatic hook5 integration

User selected0.1.86 key4 lso/nor_lso_stn_mid_trw_suc for manual tackle retrieval,
rod right, during Wait and Fight; also allow Bite. Fixed key4 starts isolated
nonloop action. At phase0.25 discard current fish without catch credit, suppress
respawn, return captured bobber/hook toward actual moving tip using smoothstep
0.25–0.85 with0.6m arc. Finish Ready, ordinary E charge/recast available. No
automatic cast. Failed startup before commit preserves current attempt.
On actual Bite->Fight via E, play chosen key5 gent/lso_stn_tug before requesting
normal fight pose. Game clock/tension/progress frozen during tug; after full
playback resume existing .65 blend and .5238 fight hold. No preview keys remain
except4 as real retrieval; no frame freezing in either action, camera free.
Existing missing-only retries/scoped ownership retained. Exact selected mappings
recorded in CONFIRMED_ITEM_ANIMATIONS.md. Portable tests cover allowed phases,
indices/source keys, single retrieve commit/no hook commit, retract endpoints.
SDK/asset checks, ZIPCRC/reproducible rebuild pass; game runtime visual validation
awaits user. Main baseline0.1.78 remains unchanged.

## 0.1.88 — fast hook-to-fight transition

User confirms0.1.87 but requests no neutral standing pause after hook. At hook
phase0.72, once fight set loaded, verify actor/session ownership and bind lso_base
while preserving active hook task. Only clear owned override flag1 if necessary;
base gent path needs no clear. Mark FightPose owned/Bound and request next frame
with0.18s blend instead of0.65s. Bypass normal-end0.25s wait and setup frames.
Fight set pre-requested at hook begin; .5238 hold and natural advancement unchanged.
Default blend retained for unrelated requests. Existing natural-end fallback if
fast handoff unavailable. Tests cover Hook-only gate and fast blend ABI. SDK,
ZIPCRC/reproducible build checks passed. Visual transition awaits user in game.
Manual retrieval4 and previous item lifecycle unchanged; main baseline0.1.78 intact.


## Основная рабочая версия0.1.88 — подтверждена пользователем

После проверки быстрого перехода подсечки в вываживание пользователь явно
назначил0.1.88 основной. Коммит сборки2e8f0d7, тег baseline-v0.1.88. Обновлены
WORKING_BASELINE.md, CONFIRMED_ANIMATION.md и CONFIRMED_ITEM_ANIMATIONS.md.
Архивы не изменены;0.1.78 и остальные предыдущие базы/теги сохранены.
Следующие эксперименты выпускать отдельно от этой подтверждённой базы.

## 0.1.89 — spin-charge distance and fish-size reel speed

User requests hold time affects cast, maximum at3+s; small faster/normal same/
large slower, then explicitly says large<=0.5. Implement1.5/1.0/0.5 by existing
weight thresholds<1/1..5/>5. Only R&&!Q positive progress multiplied; tension,
negative progress drift, probabilities/scales and animation rates unchanged.
CastCharge tracks unpaused active hold seconds capped3; raw per-frame elapsed
used for charging rather than simulation .1 clamp so3s works at low FPS. First
keydown excludes previous interval. Distance3+(CastDistance-3)*clamp(seconds/3).
Existing INI CastDistance preserved as maximum3..20, default9: tap3m,1.5s6m,3+s9m.
HUD shows power/distance. Release latches distance and camera; existing validated
water search shortens only, down to3m; flight duration from actual water target.
Tests cover monotonic charge/min/max/custom max, pause/cancel/latch, reel weight
boundaries, unchanged tension/ease, all500 seeded catches. SDK/ZIPCRC/source rebuild
byte comparison pass. Game not run in cloud. Main baseline0.1.88 unchanged.

## 0.1.90 — 2-second charge, variable reel cadence and slow empty reeling

User requests max spin2s, size click cadence, slow R reel during Wait. Explicit
follow-up sets small0.12s and normal0.18s; large0.32 and empty0.70 retained.
Charge cap/divisor2. Existing reel progress1.5/1/.5 remains. Scheduler supports
interval per call, keeps remainder for average cadence, never emits bursts.
Same explicit gain0 MENU path/five random samples and water isolation unchanged.
Wait R&&!Q adds emptyReeled0.10m/s capped actualCastRange-2; wait timer still runs,
Bite stops empty reeling. Geometry shifts Wait/Bite/Fight start consistently so
hooking doesn't teleport tackle back out. Fully empty reel ->Ready without catch.
Ready cast resets empty distance; tension and fight progress separate. HUD adds
slow-reel hint, runtime log includes click interval/state. No mesh spool animation
added; sound cadence and actual bobber/hook retrieval implement slow reel action.
Tests cover2s cap,1/5kg interval boundaries, cadence ordering, empty pause/Q stop,
bite during reeling, continuity and complete empty return/no catch.500 seeded
catches, SDK validation, ZIPCRC/reproducible rebuild pass. Runtime sound/visual
validation awaits user. Main confirmed baseline0.1.88 retained.

## 0.1.91 — tension/payout redesign, release orientation and bite cues

User requests1.7s charge; release head toward water; R only raises tension during
fish bursts, rate by size; no passive tension reset; Q pays line out with two
removed clicks; bite gets same struggle water effects/audio.
Charge divisor/cap1.7. Release-only yaw+180 from prior, verified head(-X) dot
forward at four cardinals; swim yaw untouched. Tension gain R&&pulling small.10,
normal.20,large.30/s. Idle/no-pull holds; Q priority reduces.30/s and progress.04/s.
No passive progress drift, tension0 no longer loses catch; tension1 breaks line.
Payout scheduler.22s on Q uses only DPAD_02/0FEF10E1 and NAV_01/0B2CA65B, same
MENU_INITPARAMS gain0, independent five forward clips. Gated to actual release
work using before-step tension/progress. Dat20 mappings verified read-only.
Bite onset triggers one existing Struggle cue and size splash at current bobber;
continuous Bite->Fight pull-state avoids immediate duplicate. Hook preserves cue
and ages emitter; later fight bursts unchanged. Retrieve cancels cues as before.
Invariant tests cover idle hold/rest reel/size slopes/Q rollback, zero slack,
power1.7, head orientation, payout selection, bite cue edges.500 adaptive catches
pass with budget192s (large fish slower than old48s budget). SDK/audio mappings,
ZIPCRC/reproducible rebuild pass. No game runtime; baseline0.1.88 retained.

## 0.1.92 — RedHook-only game rewards and requested mouse controls (2026-11-15)

RMB hold spins with free camera; LMB commits cast, including minimum-range direct
LMB without spin. Releasing RMB before LMB cancels to Ready/right hand. Existing
1.7s maximum charge remains. LMB hooks, Space reels, Shift pays out. E retrieves
tackle during Wait/Bite/Fight; caught E sells, F releases. Caught mouse clicks
cannot bypass the decision. Fixed layout supersedes legacy INI E/R/Q while
preserving Toggle and saved counters. Game FIRE/TARGET/JUMP/SPRINT actions scoped
through RH_ActionDisable, restored on cleanup/detach; camera axes untouched.

Actual game stat reward implementation uses RedHook GetGlobalPtr and known
Global_54086 8-byte entries in raw buffer (cash0, honor1). Guards exact supplied
EXE profile/fingerprints, writable pages, finite/range values, table header and
honor equality with independently resolved actor cache. Writes four-byte float,
reads back; cash UI/PRINT_MONEY refreshed; honor cache native uses correct
(playerSlot,float) signature, not broken one-float SDK declaration. Honor +5,
clamped +/-5000, at release impact. Sale round(weight*10), with cent-based tie
rounding (.95 ->10), at animation disappearance. Once-only catch latch; failed
preflight or write retains fish for retry and logs reason. Runtime log includes
actual before/after/delta; ordinary game save persistence awaits in-game test.

Uploaded Rampage honor routine reverse engineered: DECOR_SET_INT honor/honorAward
on temp actor followed by KILL_ACTOR_WITH_KILLER and RELEASE_ACTOR; all natives
available in RedHook, but kill approach not used to avoid kill-side effects.
Stat layout independently cross-referenced public DoctorSpechemen/RDR-AP.
No ScriptHookRDR dependency, no bundled assets or external audio.

Portable tests: mouse confirm/cancel incl simultaneous release, 961 two-decimal
prices, honor cap, failed/repeated reward transactions, prior 500 simulated
catches and mechanics. Static native ABI/cache/action-name checks, 65 SDK hashes,
11 loader exports, system DLL dependencies only. ZIP CRC and clean-source
byte-identical rebuild pass. RDR not run here; new rewards and save/load behavior
are not runtime-confirmed. Confirmed baseline 0.1.88 is preserved unchanged.

## 0.1.93 — prevent revolver draw on RMB during fishing (2026-11-15)

User reports native RMB draws revolver alongside rod in .92. Loader analysis
shows RH_ActionDisable filters script action query natives, not every native
player-controller path; previous .92 docs overstated input suppression.
Add scoped ACTOR_START_FORCE_HOLSTER 7957CA4F(actor,1,0) before Equip and maintain
force bit during fishing. ACTOR_END_FORCE_HOLSTER 231E7034 clears our lock at
cleanup/stow/emergency/off-foot/death and detach. Previously existing force
mode is preserved. No removed weapons, no hidden gun-only workaround, no repeated
holster tasks every frame, no new camera control disable. Normal draw becomes
available again after fishing. Mouse layout/rewards/animations otherwise unchanged.
Readback follows distinct actor wrapper table2C36778 and entity table2C36768,
ped+70 weapon controller, flag+48C. Native dispatch/callback ABI and bit20/40
sets/AND9F release statically verified in supplied EXE. Runtime logs force
readback/ownership changes. Tests cover 1000-frame non-repetition, lost lock,
existing foreign mode, repeated cleanup, actor switch, unreadable state; prior
mechanics/catch tests still pass. 67 SDK natives, 11 exact loader exports;
ZIP CRC and clean-source byte-identical rebuild pass. In-game verification
awaits user. Confirmed baseline0.1.88 preserved.

## 0.1.94 — unblock sale/release from supplied failure log (2026-11-15)

User log .93: forceHolstered=1 readback confirmed; every reward rejected at
`honor table/cache mismatch or player not ready`. All prior profile/table/range/
writability gates had passed. Remove the combined optional native/HUD cache
eligibility gate, which wrongly blocked both money and honor decisions. Preserve
actual persistent stat write/readback and all earlier guards. Cache remains
read-only diagnostic: readiness/readability/cacheHonor plus actual cash/honor
now logged separately at preflight. No invented success on failed write.
Independent Rampage honor getter reads Global54086+10 hex and cash getter+08,
confirming stat entry layout already cross-referenced from RDR-AP. Native cache
refresh safely checks readiness itself. No other mechanic/control/weapon change.
Portable regression suite, SDK/native ABI/weapon flag static checks, ZIP CRC,
clean-source reproducible build pass. Actual reward outcomes/save-load still
await user test; no RDR runtime available in workspace. Baseline0.1.88 preserved.

## 0.1.95 — native honor HUD, longer calm windows, 10m casts (2026-11-15)

User's .94 logs confirm honor at cap5000 ->5000 delta0; subsequent trainer test
4950 ->4955 delta5. Thus actual +5 works; missing native presentation addressed
with SET_STAT_MESSAGE 73DA6AF1, game sn_hnrr label/hash AFA3172F, actual delta,
3s, type2/honor selector0 following game's stat-popup pattern. Existing native
honor value refresh precedes HUD notification. No custom honor meter; own help/
corner honor text removed per user's explicit correction. Capped gains produce
no fake increase or raw cache write. Native HUD appearance requires visual test.

Burst duration3.2s unchanged; calm window1.6 ->4.8s (8s cycle). Shared fishPulling
helper drives tension/motion/water audio/VFX/HUD; old duplicate HUD clock removed.
Tests cover complete calm reel interval without added tension, burst boundaries,
no water struggle sound during extended rest and next burst at8s. Existing
adaptive500 catches still pass. Cast max10m after user corrected19 typo; default
10 and old INI default9 mapped10 without replacing file/counters; upper bound10.
1.7s full charge retained; 0/.85/1.7s ->3/6.5/10m verified.
68 SDK hashes,11 exact RedHook exports; ABI/weapon checks, ZIP CRC, clean-source
byte-identical rebuild pass. No cloud game execution; confirmed main baseline
0.1.88 preserved, new UI/rhythm await user validation.

Final follow-up before handing over .95: random bite wait7..15 seconds, replacing
4..10.1000 random rolls validate bounds/end coverage;500 catch regressions and
clean-source identical rebuild pass. Same .95 archives updated before final link.
