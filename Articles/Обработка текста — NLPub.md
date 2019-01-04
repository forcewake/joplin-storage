Обработка текста — NLPub

# Обработка текста

Материал из NLPub

Перейти к: [навигация](https://nlpub.ru/#mw-head), [поиск](https://nlpub.ru/#p-search)

## Содержание

 \[убрать\] 

*   [1 Графематический анализ](https://nlpub.ru/#.D0.93.D1.80.D0.B0.D1.84.D0.B5.D0.BC.D0.B0.D1.82.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.B8.D0.B9_.D0.B0.D0.BD.D0.B0.D0.BB.D0.B8.D0.B7)
*   [2 Морфологический анализ](https://nlpub.ru/#.D0.9C.D0.BE.D1.80.D1.84.D0.BE.D0.BB.D0.BE.D0.B3.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.B8.D0.B9_.D0.B0.D0.BD.D0.B0.D0.BB.D0.B8.D0.B7)
*   [3 Синтаксический анализ](https://nlpub.ru/#.D0.A1.D0.B8.D0.BD.D1.82.D0.B0.D0.BA.D1.81.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.B8.D0.B9_.D0.B0.D0.BD.D0.B0.D0.BB.D0.B8.D0.B7)
*   [4 Проверка правописания](https://nlpub.ru/#.D0.9F.D1.80.D0.BE.D0.B2.D0.B5.D1.80.D0.BA.D0.B0_.D0.BF.D1.80.D0.B0.D0.B2.D0.BE.D0.BF.D0.B8.D1.81.D0.B0.D0.BD.D0.B8.D1.8F)
*   [5 Расстановка переносов](https://nlpub.ru/#.D0.A0.D0.B0.D1.81.D1.81.D1.82.D0.B0.D0.BD.D0.BE.D0.B2.D0.BA.D0.B0_.D0.BF.D0.B5.D1.80.D0.B5.D0.BD.D0.BE.D1.81.D0.BE.D0.B2)
*   [6 Построение конкордансов](https://nlpub.ru/#.D0.9F.D0.BE.D1.81.D1.82.D1.80.D0.BE.D0.B5.D0.BD.D0.B8.D0.B5_.D0.BA.D0.BE.D0.BD.D0.BA.D0.BE.D1.80.D0.B4.D0.B0.D0.BD.D1.81.D0.BE.D0.B2)
*   [7 Извлечение ключевых слов](https://nlpub.ru/#.D0.98.D0.B7.D0.B2.D0.BB.D0.B5.D1.87.D0.B5.D0.BD.D0.B8.D0.B5_.D0.BA.D0.BB.D1.8E.D1.87.D0.B5.D0.B2.D1.8B.D1.85_.D1.81.D0.BB.D0.BE.D0.B2)
*   [8 Автоматическое реферирование](https://nlpub.ru/#.D0.90.D0.B2.D1.82.D0.BE.D0.BC.D0.B0.D1.82.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.BE.D0.B5_.D1.80.D0.B5.D1.84.D0.B5.D1.80.D0.B8.D1.80.D0.BE.D0.B2.D0.B0.D0.BD.D0.B8.D0.B5)
*   [9 Тематическая классификация](https://nlpub.ru/#.D0.A2.D0.B5.D0.BC.D0.B0.D1.82.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.B0.D1.8F_.D0.BA.D0.BB.D0.B0.D1.81.D1.81.D0.B8.D1.84.D0.B8.D0.BA.D0.B0.D1.86.D0.B8.D1.8F)
*   [10 Тематическое моделирование](https://nlpub.ru/#.D0.A2.D0.B5.D0.BC.D0.B0.D1.82.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.BE.D0.B5_.D0.BC.D0.BE.D0.B4.D0.B5.D0.BB.D0.B8.D1.80.D0.BE.D0.B2.D0.B0.D0.BD.D0.B8.D0.B5)
*   [11 Извлечение именованных сущностей](https://nlpub.ru/#.D0.98.D0.B7.D0.B2.D0.BB.D0.B5.D1.87.D0.B5.D0.BD.D0.B8.D0.B5_.D0.B8.D0.BC.D0.B5.D0.BD.D0.BE.D0.B2.D0.B0.D0.BD.D0.BD.D1.8B.D1.85_.D1.81.D1.83.D1.89.D0.BD.D0.BE.D1.81.D1.82.D0.B5.D0.B9)
*   [12 Извлечение отношений](https://nlpub.ru/#.D0.98.D0.B7.D0.B2.D0.BB.D0.B5.D1.87.D0.B5.D0.BD.D0.B8.D0.B5_.D0.BE.D1.82.D0.BD.D0.BE.D1.88.D0.B5.D0.BD.D0.B8.D0.B9)
*   [13 Анализ тональности](https://nlpub.ru/#.D0.90.D0.BD.D0.B0.D0.BB.D0.B8.D0.B7_.D1.82.D0.BE.D0.BD.D0.B0.D0.BB.D1.8C.D0.BD.D0.BE.D1.81.D1.82.D0.B8)
*   [14 Информационный поиск](https://nlpub.ru/#.D0.98.D0.BD.D1.84.D0.BE.D1.80.D0.BC.D0.B0.D1.86.D0.B8.D0.BE.D0.BD.D0.BD.D1.8B.D0.B9_.D0.BF.D0.BE.D0.B8.D1.81.D0.BA)
*   [15 Машинный перевод](https://nlpub.ru/#.D0.9C.D0.B0.D1.88.D0.B8.D0.BD.D0.BD.D1.8B.D0.B9_.D0.BF.D0.B5.D1.80.D0.B5.D0.B2.D0.BE.D0.B4)
*   [16 Обнаружение дубликатов](https://nlpub.ru/#.D0.9E.D0.B1.D0.BD.D0.B0.D1.80.D1.83.D0.B6.D0.B5.D0.BD.D0.B8.D0.B5_.D0.B4.D1.83.D0.B1.D0.BB.D0.B8.D0.BA.D0.B0.D1.82.D0.BE.D0.B2)
*   [17 Cегментация текста](https://nlpub.ru/#C.D0.B5.D0.B3.D0.BC.D0.B5.D0.BD.D1.82.D0.B0.D1.86.D0.B8.D1.8F_.D1.82.D0.B5.D0.BA.D1.81.D1.82.D0.B0)
*   [18 Интегрированные пакеты](https://nlpub.ru/#.D0.98.D0.BD.D1.82.D0.B5.D0.B3.D1.80.D0.B8.D1.80.D0.BE.D0.B2.D0.B0.D0.BD.D0.BD.D1.8B.D0.B5_.D0.BF.D0.B0.D0.BA.D0.B5.D1.82.D1.8B)
*   [19 Примечания](https://nlpub.ru/#.D0.9F.D1.80.D0.B8.D0.BC.D0.B5.D1.87.D0.B0.D0.BD.D0.B8.D1.8F)

## [Графематический анализ](https://nlpub.ru/%D0%93%D1%80%D0%B0%D1%84%D0%B5%D0%BC%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B9_%D0%B0%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7 "Графематический анализ")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [АОТ](https://nlpub.ru/%D0%90%D0%9E%D0%A2#.D0.93.D1.80.D0.B0.D1.84.D0.B5.D0.BC.D0.B0.D1.82.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.B8.D0.B9_.D0.B0.D0.BD.D0.B0.D0.BB.D0.B8.D0.B7 "АОТ") | словарный | русский, английский | LGPL | Linux, Windows |
| [Lemmatizer](http://lemmatizer.org) | словарный | русский, английский | GPL | Linux |
| [FreeLing](https://nlpub.ru/FreeLing "FreeLing") | правила | русский, английский, итальянский, испанский, португальский, астурийский, валийский, галисийский, каталанский | GPL + Коммерческая | Linux |
| [Stanford CoreNLP](https://nlpub.ru/Stanford_CoreNLP "Stanford CoreNLP") | эвристика | английский | GPL | Java |
| [Apache OpenNLP](https://nlpub.ru/Apache_OpenNLP "Apache OpenNLP") | регулярные выражения, машинное обучение | английский | Apache License | Java |
| [Twitter NLP and Part-of-Speech Tagger](http://www.ark.cs.cmu.edu/TweetNLP/) | машинное обучение | английский | GPL | Java |
| [NLTK](http://nltk.org) | регулярные выражения, машинное обучение | английский | Apache License | Python |
| [TextBlob](https://textblob.readthedocs.org/) | регулярные выражения, машинное обучение | английский | MIT | Python |
| [MBSP](http://www.clips.ua.ac.be/pages/MBSP) | машинное обучение | английский | GPL | Python |
| [Pattern](http://www.clips.ua.ac.be/pattern) | правила, регулярные выражения | английский, испанский, немецкий, французский, итальянский, нидерландский | BSD | Python |
| [Greeb](https://nlpub.ru/Greeb "Greeb") | регулярные выражения | русский, английский | MIT | Ruby |
| [natural](https://github.com/NaturalNode/natural) | регулярные выражения | английский, испанский, персидский, итальянский, русский | MIT | Node.js |
| [Solarix](http://solarix.ru/for_developers/docs/tokenizer.shtml) | правила | русский, английский | Коммерческая | Linux, Windows |
| [tokenizer](http://www.cis.uni-muenchen.de/~wastl/misc/) | правила | русский, английский, немецкий | GPL | C   |
| [AskNet](http://www.asknet.ru/Technology/techdescr.htm) | правила | русский, английский | Коммерческая | Windows |

## [Морфологический анализ](https://nlpub.ru/%D0%9C%D0%BE%D1%80%D1%84%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B9_%D0%B0%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7 "Морфологический анализ")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [АОТ](https://nlpub.ru/%D0%90%D0%9E%D0%A2#.D0.9C.D0.BE.D1.80.D1.84.D0.BE.D0.BB.D0.BE.D0.B3.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.B8.D0.B9_.D0.B0.D0.BD.D0.B0.D0.BB.D0.B8.D0.B7 "АОТ") | словарный | русский, английский, немецкий | LGPL | Linux, Windows |
| [Snowball](https://nlpub.ru/Snowball "Snowball") | алгоритм Портера | русский, английский | BSD | Linux, Windows |
| [Stemka](https://nlpub.ru/Stemka "Stemka") | словарный | русский | Собственная | Linux, Windows |
| [pymorphy](https://nlpub.ru/Pymorphy "Pymorphy") | словарный | русский, английский, немецкий | MIT | Python |
| [Myaso](https://nlpub.ru/Myaso "Myaso") | [алгоритм Витерби](https://nlpub.ru/%D0%90%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC_%D0%92%D0%B8%D1%82%D0%B5%D1%80%D0%B1%D0%B8 "Алгоритм Витерби") | русский, английский | MIT | Ruby |
| [Eureka Engine](https://nlpub.ru/Eureka_Engine "Eureka Engine") | машинное обучение | русский | Коммерческая | Веб-сервис |
| [ISPRAS API Texterra](https://api.ispras.ru/) | машинное обучение | русский, английский | Бесплатная для исследовательских целей \+ коммерческая | Веб-сервис, Java, Python |
| [pymystem3](https://pypi.python.org/pypi/pymystem3) | [разрешение омонимии](http://download.yandex.ru/company/iseg-las-vegas.pdf) | русский, английский | LGPLv3 + некоммерческая | Python, C++ |
| [phpmorphy](http://phpmorphy.sourceforge.net/dokuwiki/) | словарный | русский, английский, немецкий | LGPL | PHP |
| [Pullenti SDK](http://pullenti.ru) | словарный | русский, английский, украинский | Non-Commercial Freeware | .NET, .NET Core, Java и Python |
| [FreeLing](https://nlpub.ru/FreeLing "FreeLing") | словарный | русский, англиский, итальянский, испанский, португальский, астурийский, валийский, галисийский, каталанский | GPL + Коммерческая | Linux |
| [NLTK](http://nltk.org) | машинное обучение | английский | Apache License | Python |
| [TextBlob](https://textblob.readthedocs.org/) | машинное обучение | английский | MIT | Python |
| [MBSP](http://www.clips.ua.ac.be/pages/MBSP) | машинное обучение | английский | GPL | Python |
| [Pattern](http://www.clips.ua.ac.be/pattern) | правила, регулярные выражения | английский, испанский, немецкий, французский, итальянский, нидерландский | BSD | Python |
| [natural](https://github.com/NaturalNode/natural) | правила | английский, французский, японский | MIT | Node.js |
| [MAnalyzer](https://github.com/melkogotto/MAnalyzer) | словарный | русский, английский | MIT | Linux |
| [hunpos](https://code.google.com/p/hunpos/) | [алгоритм Витерби](https://nlpub.ru/%D0%90%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC_%D0%92%D0%B8%D1%82%D0%B5%D1%80%D0%B1%D0%B8 "Алгоритм Витерби") | английский, корейский | BSD | Linux, Windows |
| [SVMTool](http://www.lsi.upc.edu/~nlp/SVMTool/) | метод опорных векторов | русский, английский | LGPL | Perl |
| [Twitter NLP and Part-of-Speech Tagger](http://www.ark.cs.cmu.edu/TweetNLP/) | машинное обучение | английский | GPL | Java |
| [Stanford Log-linear Part-Of-Speech Tagger](http://nlp.stanford.edu/software/tagger.shtml) | машинное обучение | английский, немецкий, арабский, китайский | GPL | Java |
| [RussianMorphology](http://code.google.com/p/russianmorphology/) | словарный | русский | Apache License | Java |
| [RussianPOSTagger](http://rupostagger.sourceforge.net/) | словарный | русский | GPL | Java |
| [mystem](https://nlpub.ru/Mystem "Mystem") | словарный | русский | Некоммерческая | Linux, Windows |
| [TreeTagger](https://nlpub.ru/TreeTagger "TreeTagger") | деревья принятия решений | русский, английский, немецкий, французский, итальянский, нидерландский, испанский, болгарский, греческий, португальский, китайский, суахили, латинский, эстонский | Некоммерческая | Linux, Windows |
| [TnT](http://www.coli.uni-saarland.de/~thorsten/tnt/) | [алгоритм Витерби](https://nlpub.ru/%D0%90%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC_%D0%92%D0%B8%D1%82%D0%B5%D1%80%D0%B1%D0%B8 "Алгоритм Витерби") | русский, английский | Некоммерческая | Linux |
| [Морфер](http://morpher.ru) | словарный | русский, украинский | Коммерческая | Windows, Веб-сервис |
| [RCO](http://rco.ru/product.asp?ob_no=2871) | словарный | русский | Коммерческая | Windows |
| [AskNet](http://www.asknet.ru/Technology/morpho.htm) | словарный, правила | русский, английский | Коммерческая | Windows |
| [Solarix](http://solarix.ru/for_developers/api/morphology-analyzer-api.shtml) | словарный | русский, английский | Коммерческая | Linux, Windows |
| [ОРФО](http://www.informatic.ru/catalogue/3) | словарный | русский, украинский, английский, французский, немецкий, испанский, итальянский, португальский | Коммерческая | Windows |
| [STARLING](http://starling.rinet.ru/downl.php?lan=ru#soft) | словарный | русский | н/д | Windows |
| [mystem-scala](https://github.com/alexeyev/mystem-scala) | словарный | русский | MIT + некоммерческая | Java on Linux, Windows |
| [zamgi](https://github.com/zamgi/lingvo--PosTagger-ru) | машинное обучение, словарный | русский | некоммерческая | .NET on Linux, Windows |
| [zamgi](https://github.com/zamgi/lingvo--PosTagger-en) | машинное обучение, словарный | английский | некоммерческая | .NET on Linux, Windows |

## [Синтаксический анализ](https://nlpub.ru/%D0%A1%D0%B8%D0%BD%D1%82%D0%B0%D0%BA%D1%81%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B9_%D0%B0%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7 "Синтаксический анализ")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [АОТ](https://nlpub.ru/%D0%90%D0%9E%D0%A2#.D0.A1.D0.B8.D0.BD.D1.82.D0.B0.D0.BA.D1.81.D0.B8.D1.87.D0.B5.D1.81.D0.BA.D0.B8.D0.B9_.D0.B0.D0.BD.D0.B0.D0.BB.D0.B8.D0.B7 "АОТ") | [грамматика HPSG](https://nlpub.ru/Head-Driven_Phrase_Structure_Grammar "Head-Driven Phrase Structure Grammar") | русский, английский, немецкий | LGPL | Linux, Windows |
| [ISPRAS API Texterra](https://api.ispras.ru/) | машинное обучение | русский, английский | Бесплатная для исследовательских целей \+ коммерческая | Веб-сервис, Java, Python |
| [MaltParser](https://nlpub.ru/MaltParser "MaltParser") | машинное обучение | русский, английский | [Собственная](http://www.maltparser.org/license.html) | Java |
| [MSTParser](http://sourceforge.net/projects/mstparser/) | максимизация остовного дерева | английский, португальский | Apache License | Java |
| [Link Grammar Parser](https://nlpub.ru/Link_Grammar_Parser "Link Grammar Parser") | грамматика связей | русский, английский | BSD | Linux, Windows |
| [AGFL](http://www.agfl.cs.ru.nl/) | грамматика аффиксов над конечной решёткой | русский[\[1\]](https://nlpub.ru/#cite_note-1), английский, французский, испанский, арабский | GPL | Linux, Windows |
| [NLTK](http://nltk.org) | машинное обучение | английский | Apache License | Python |
| [MBSP](http://www.clips.ua.ac.be/pages/MBSP) | машинное обучение | английский | GPL | Python |
| [Pattern](http://www.clips.ua.ac.be/pattern) | правила, регулярные выражения | английский, испанский, немецкий, французский, итальянский, нидерландский | BSD | Python |
| [Solarix](http://solarix.ru/for_developers/api/syntax-analyzer-api.shtml) | правила | русский, английский | Коммерческая | Linux, Windows |
| [ABBYY Compreno](http://www.abbyy.ru) | правила | русский | Коммерческая | Windows |
| [AskNet](http://www.asknet.ru/Technology/syntax.htm) | правила | русский, английский | Коммерческая | Windows |
| [DictaScope](http://www.dictum.ru/) | правила | русский | Коммерческая | FreeBSD, Windows |
| [ЭТАП-3](http://www.iitp.ru/ru/science/works/452.htm) | правила | русский, английский | н/д | Windows |
| [Синтактико-Семантический Анализ Русского Языка](http://semanticanalyzer.info/blog/demo/) | функциональная модель языка | русский | Коммерческая | Веб-сервис |
| [The Stanford Parser](http://nlp.stanford.edu/software/lex-parser.shtml) | машинное обучение | английский, немецкий, арабский, китайский, болгарский, итальянский, португальский | GPL | Java |
| [ZPar](http://www.sutd.edu.sg/cmsresource/faculty/yuezhang/zpar.html) | машинное обучение | английский, китайский, румынский | GPL v3 | C++ |
| [mate-tools](http://code.google.com/p/mate-tools/) | машинное обучение | английский, немецкий, китайский, испанский | GPL v2 | Java |
| [zamgi](https://github.com/zamgi/lingvo--Syntax-ru) | машинное обучение | русский | некоммерческая | .NET on Linux, Windows |

## [Проверка правописания](https://nlpub.ru/%D0%9F%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%BA%D0%B0_%D0%BF%D1%80%D0%B0%D0%B2%D0%BE%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D1%8F "Проверка правописания")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [GNU Aspell](http://aspell.net) | н/д | более 70 языков, в том числе русский и английский | LGPL | Linux, Windows |
| [Hunspell](http://hunspell.sourceforge.net) | н/д | более 100 языков, в том числе русский и английский | GPL, LGPL, MPL | Linux, Windows, OS X |
| [Pattern](http://www.clips.ua.ac.be/pattern) | правила, регулярные выражения | английский, испанский, немецкий, французский, итальянский, нидерландский | BSD | Python |
| [Spellah](https://nlpub.ru/Spellah "Spellah") | n-граммы поисковой машины | английский | MIT | Веб-сервис, Node.js |
| [Yandex speller](https://tech.yandex.ru/speller/) | орфографический словарь | русский, украинский, английский | своя | Веб-сервис |
| [ОРФО Speller](http://www.informatic.ru/catalogue/1) | н/д | русский, украинский, английский, французский, немецкий, испанский, итальянский, португальский | Коммерческая | Windows |
| [ОРФО Grammar Checker](http://www.informatic.ru/catalogue/2) | н/д | русский | Коммерческая | Windows |
| [Орфограммка](http://orfogrammka.ru) | правила, словарный, машинное обучение | русский, английский, латинский | Коммерческая | Веб-сервис |
| [LanguageTool](http://www.languagetool.org) | правила | английский, русский | LGPL | Java |

## [Расстановка переносов](https://nlpub.ru/%D0%A0%D0%B0%D1%81%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0_%D0%BF%D0%B5%D1%80%D0%B5%D0%BD%D0%BE%D1%81%D0%BE%D0%B2 "Расстановка переносов")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Text::Hyphen](http://rubygems.org/gems/text-hyphen) | шаблоны переносов TeX | более 30 языков, в том числе русский и английский | MIT | Ruby |
| [ОРФО](http://www.informatic.ru/catalogue/5) | н/д | русский | Коммерческая | Windows |

## [Построение конкордансов](https://nlpub.ru/%D0%9F%D0%BE%D1%81%D1%82%D1%80%D0%BE%D0%B5%D0%BD%D0%B8%D0%B5_%D0%BA%D0%BE%D0%BD%D0%BA%D0%BE%D1%80%D0%B4%D0%B0%D0%BD%D1%81%D0%BE%D0%B2 "Построение конкордансов")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Iramuteq](http://www.iramuteq.org/) | иерархическая нисходящая классификация | французский | GPL | R и Python |
| [NooJ](http://www.nooj4nlp.net/pages/nooj.html) | н/д | н/д | AGPL | Java |
| [Alceste](http://www.image-zafar.com/en/alceste-software) | иерархическая нисходящая классификация | французский, английский, испанский, немецкий, итальянский, португальский | Коммерческая | Windows |

## [Извлечение ключевых слов](https://nlpub.ru/%D0%98%D0%B7%D0%B2%D0%BB%D0%B5%D1%87%D0%B5%D0%BD%D0%B8%D0%B5_%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%B2%D1%8B%D1%85_%D1%81%D0%BB%D0%BE%D0%B2 "Извлечение ключевых слов")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Kea](http://www.nzdl.org/Kea/) | Kea | английский | GPL | Java |
| [Maui](http://code.google.com/p/maui-indexer/) | Kea + Wikipedia | английский | GPL | Java |
| [Tesuck](https://nlpub.ru/Tesuck "Tesuck") | [DegExt](https://nlpub.ru/index.php?title=DegExt&action=edit&redlink=1 "DegExt (страница не существует)"), [TextRank](https://nlpub.ru/TextRank "TextRank") | русский, английский | Некоммерческая | Веб-сервис |
| [TextMF](https://bitbucket.org/b0noI/textmf) | частотный анализ | русский, английский | н/д | Java |
| [natural](https://github.com/NaturalNode/natural) | TF-IDF | английский | MIT | Node.js |
| [Content Analyzer](http://www.rvsn2.narod.ru/soft51.htm) | TF-IDF | русский | Freeware | Windows |
| [AskNet](http://www.asknet.ru/Technology/dictionary.htm) | TF-IDF, словари, правила | русский, английский | Коммерческая | Windows |
| [TextAnalyst](http://analyst.ru/index.php?lang=eng&dir=content/tech/&id=approach) | нейронная сеть | русский | Коммерческая | Windows |
| [AlchemyAPI](http://www.alchemyapi.com) | н/д | английский | Коммерческая | Веб-сервис |
| [Семантическое зеркало](http://www.ashmanov.com/tech/semantic) | н/д | русский | Коммерческая | Веб-сервис |
| [Extractor](http://extractorlive.com) | генетический алгоритм | английский, французский, японский, немецкий, испанский, корейский | Коммерческая | Веб-сервис |
| [TerMine](http://www.nactem.ac.uk/software/termine/) | C-value | английский | Коммерческая | Веб-сервис |

## [Автоматическое реферирование](https://nlpub.ru/%D0%90%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5_%D1%80%D0%B5%D1%84%D0%B5%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5 "Автоматическое реферирование")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Open Text Summarizer](http://libots.sourceforge.net/) | частоты, правила | русский, английский, немецкий, испанский, иврит, эсперанто | GPL | Linux, Windows |
| [SummarizeBot](https://www.summarizebot.com/) | машинное обучение, блокчейн | русский, английский, китайский, японский и 100+ других языков | Proprietary | Веб-сервис (чат-бот) |
| [SweSum](http://swesum.nada.kth.se/) | частоты | шведский, датский, норвежский, английский | н/д | Веб-сервис |
| [Content Analyzer](http://www.rvsn2.narod.ru/soft51.htm) | TF-IDF | русский | Freeware | Windows |
| [Tesuck](https://nlpub.ru/Tesuck "Tesuck") | [TextRank](https://nlpub.ru/TextRank "TextRank"), [DegExt](https://nlpub.ru/index.php?title=DegExt&action=edit&redlink=1 "DegExt (страница не существует)") | русский, английский | Некоммерческая | Веб-сервис |
| [Extractor](http://extractorlive.com) | генетический алгоритм | английский, французский, японский, немецкий, испанский, корейский | Коммерческая | Веб-сервис |
| [Рефератор](https://visualworld.ru/referat.jsp) | частоты, правила | русский, английский | Некоммерческая | Веб-сервис |

## [Тематическая классификация](https://nlpub.ru/%D0%A2%D0%B5%D0%BC%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F_%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F "Тематическая классификация")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Maui](http://code.google.com/p/maui-indexer/) | Kea + Wikipedia | английский | GPL | Java |
| [Eureka Engine](https://nlpub.ru/Eureka_Engine "Eureka Engine") | метод опорных векторов | русский | Коммерческая | Веб-сервис |
| [TextMF](https://bitbucket.org/b0noI/textmf) | частотный анализ | русский, английский | н/д | Java |
| [Семантическое зеркало](http://www.ashmanov.com/tech/semantic) | н/д | русский | Коммерческая | Веб-сервис |
| [AlchemyAPI](http://www.alchemyapi.com) | н/д | английский | Коммерческая | Веб-сервис |
| [zamgi](https://github.com/zamgi/lingvo--classify) | SVM | русский | некоммерческая | .NET on Linux, Windows |

## [Тематическое моделирование](https://nlpub.ru/%D0%A2%D0%B5%D0%BC%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5_%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5 "Тематическое моделирование")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [MALLET](https://nlpub.ru/MALLET "MALLET") | латентное размещение Дирихле | русский, английский | CPL | Java |
| [BigARTM](https://nlpub.ru/BigARTM "BigARTM") | аддитивная регуляризация тематических моделей | русский, английский | BSD | Windows, Linux, OS X |
| [Gensim](https://nlpub.ru/Gensim "Gensim") | латентное размещение Дирихле, латентный семантический анализ | русский, английский | LGPL | Python |
| [Weka](https://nlpub.ru/index.php?title=Weka&action=edit&redlink=1 "Weka (страница не существует)") | EM-алгоритм | русский, английский | GPL + некоммерческая | Java |
| [Insider](http://semanticanalyzer.info/blog/insider-api-trend-search/) | realtime кластеризация поисковой выдачи | русский, английский, китайский | коммерческая | JSON API |

## [Извлечение именованных сущностей](https://nlpub.ru/%D0%98%D0%B7%D0%B2%D0%BB%D0%B5%D1%87%D0%B5%D0%BD%D0%B8%D0%B5_%D0%B8%D0%BC%D0%B5%D0%BD%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D1%8B%D1%85_%D1%81%D1%83%D1%89%D0%BD%D0%BE%D1%81%D1%82%D0%B5%D0%B9 "Извлечение именованных сущностей")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [FreeLing](https://nlpub.ru/FreeLing "FreeLing") | конечный автомат | русский, английский, итальянский, испанский, португальский, астурийский, валийский, галисийский, каталанский | GPL + Коммерческая | Linux |
| [OpenCalais](http://www.opencalais.com) | н/д | английский | Некоммерческая/Коммерческая | Веб-сервис |
| [AlchemyAPI](http://www.alchemyapi.com) | н/д | английский | Коммерческая | Веб-сервис |
| [Eureka Engine](https://nlpub.ru/Eureka_Engine "Eureka Engine") | условные случайные поля | русский, английский | Коммерческая | Веб-сервис |
| [ISPRAS API Texterra](https://api.ispras.ru/) | машинное обучение | русский, английский | Бесплатная для исследовательских целей \+ коммерческая | Веб-сервис, Java, Python |
| [DBPedia Spotlight](https://github.com/dbpedia-spotlight/dbpedia-spotlight) | н/д | английский | Apache License | Java, Scala, Веб-сервис |
| [Yahoo! Content Analysis](http://developer.yahoo.com/search/content/V2/contentAnalysis.html) | н/д | английский | Коммерческая | Веб-сервис |
| [CiceroLite](http://www.languagecomputer.com/products/text-annotation/cicerolite.html) | н/д | английский, арабский, китайский и др. | Коммерческая | Linux, Windows, OS X, Solaris, Веб-сервис |
| [Stanford NER](http://nlp.stanford.edu/software/CRF-NER.shtml) | машинное обучение (Conditional Random Field sequence models) | английский, немецкий | GPL | Linux, Windows, OS X |
| [Apache cTAKES](http://ctakes.apache.org/) | правила, машинное обучение | английский | Apache License | Java |
| [TextMF](https://bitbucket.org/b0noI/textmf) | частотный анализ | русский, английский | н/д | Java |
| [PullEnti SDK](http://pullenti.ru) | правила | русский, украинский, английский | Non-Commercial Freeware | .NET, .NET Core, Java и Python |
| [LingPipe](http://alias-i.com/lingpipe/) | машинное обучение | английский | Коммерческая | Java |
| [Томита-парсер](https://nlpub.ru/%D0%A2%D0%BE%D0%BC%D0%B8%D1%82%D0%B0-%D0%BF%D0%B0%D1%80%D1%81%D0%B5%D1%80 "Томита-парсер") | [словари](https://nlpub.ru/%D0%A1%D0%BB%D0%BE%D0%B2%D0%B0%D1%80%D0%B8 "Словари") и [контекстно-свободные грамматики](https://nlpub.ru/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%BD%D0%BE-%D1%81%D0%B2%D0%BE%D0%B1%D0%BE%D0%B4%D0%BD%D1%8B%D0%B5_%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B0%D1%82%D0%B8%D0%BA%D0%B8 "Контекстно-свободные грамматики") | русский | [Особая](https://nlpub.ru/%D0%A2%D0%BE%D0%BC%D0%B8%D1%82%D0%B0-%D0%BF%D0%B0%D1%80%D1%81%D0%B5%D1%80#.D0.94.D0.BE.D1.81.D1.82.D1.83.D0.BF.D0.BD.D0.BE.D1.81.D1.82.D1.8C "Томита-парсер") | Linux, Windows, OS X |
| [TEXToCAT](http://textocat.com/) | машинное обучение | русский | Коммерческая | Веб-сервис |
| [RCO Fact Extractor SDK](http://www.rco.ru/product.asp?ob_no=5047) | правила | русский | Коммерческая | Linux, Windows |
| [OntosMiner](http://www.ontos.com/products/ontosminer/) | н/д | русский, английский, французский, немецкий | Коммерческая | Java |
| [X-Files](http://x-file.su/ner/) | н/д | русский, английский | Коммерческая | Веб-сервис |
| [AskNet](http://www.asknet.ru/Technology/semantic.htm) | н/д | русский, английский | Коммерческая | Windows, C++ |
| [ABBYY Intelligent Tagger](http://www.abbyy.ru/itagger/) | н/д | русский | Коммерческая | .NET |
| [NetOwl Extractor](https://www.netowl.com/entity-extraction/) | н/д | русский, английский, арабский, китайский, французский, немецкий, корейский, персидский, испанский | Коммерческая | Linux, Windows |
| [ИАС "АРИОН"](http://www.sytech.ru/about.php?id=149) | н/д | русский | Коммерческая | Java |
| [МетаФраз](http://www.metafraz.ru/index/0-4) | н/д | русский | Коммерческая | Windows |
| [DictaScope Tokenizer](http://dictum.ru/ru/named-entities-extraction/blog) | н/д | русский | Коммерческая | FreeBSD, Windows |
| [XANALYS Indexer](http://www.xanalys.ru/?p=SolutionsIndexer) | н/д | русский | Коммерческая | н/д |
| [Rosette](http://www.basistech.com/text-analytics/rosette/) | н/д | русский, английский | Коммерческая | н/д |
| [Natasha](https://github.com/bureaucratic-labs/natasha) | правила, машинное обучение | русский, английский (частично) | MIT | Python |
| [zamgi](https://github.com/zamgi/lingvo--Ner-ru) | машинное обучение | русский | некоммерческая | .NET on Linux, Windows |
| [zamgi](https://github.com/zamgi/lingvo--Ner-en) | машинное обучение | английский | некоммерческая | .NET on Linux, Windows |

## [Извлечение отношений](https://nlpub.ru/%D0%98%D0%B7%D0%B2%D0%BB%D0%B5%D1%87%D0%B5%D0%BD%D0%B8%D0%B5_%D0%BE%D1%82%D0%BD%D0%BE%D1%88%D0%B5%D0%BD%D0%B8%D0%B9 "Извлечение отношений")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Serelex](https://github.com/cental/PatternSim) | Лексико-синтаксические шаблоны. [демо](http://serelex.cental.be) | английский, французский, русский | LGPL | Linux, Windows, OS X |
| [ReVerb](http://reverb.cs.washington.edu/) | машинное обучение | английский | Некоммерческая | Java |
| [RCO](http://rco.ru/technology.asp?ob_no=1920) | н/д | русский | Коммерческая | Windows |
| [AskNet](http://www.asknet.ru/Technology/syntax.htm) | словари, правила | русский, английский | Коммерческая | Windows, C++ |
| [AlchemyAPI](http://www.alchemyapi.com) | н/д | английский | Коммерческая | Веб-сервис |
| [OpenCalais](http://www.opencalais.com) | н/д | английский | Коммерческая | Веб-сервис |
| [Томита-парсер](https://nlpub.ru/%D0%A2%D0%BE%D0%BC%D0%B8%D1%82%D0%B0-%D0%BF%D0%B0%D1%80%D1%81%D0%B5%D1%80 "Томита-парсер") | [словари](https://nlpub.ru/%D0%A1%D0%BB%D0%BE%D0%B2%D0%B0%D1%80%D0%B8 "Словари") и [контекстно-свободные грамматики](https://nlpub.ru/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%BD%D0%BE-%D1%81%D0%B2%D0%BE%D0%B1%D0%BE%D0%B4%D0%BD%D1%8B%D0%B5_%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B0%D1%82%D0%B8%D0%BA%D0%B8 "Контекстно-свободные грамматики") | русский | [Особая](https://nlpub.ru/%D0%A2%D0%BE%D0%BC%D0%B8%D1%82%D0%B0-%D0%BF%D0%B0%D1%80%D1%81%D0%B5%D1%80#.D0.94.D0.BE.D1.81.D1.82.D1.83.D0.BF.D0.BD.D0.BE.D1.81.D1.82.D1.8C "Томита-парсер") | Linux, Windows, OS X |
| [OntosMiner](http://www.ontos.com/products/ontosminer/) | н/д | русский, английский, французский, немецкий | Коммерческая | Java |
| [NetOwl Extractor](https://www.netowl.com/entity-extraction/) | н/д | русский, английский, арабский, китайский, французский, немецкий, корейский, персидский, испанский | Коммерческая | Linux, Windows |

## [Анализ тональности](https://nlpub.ru/%D0%90%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7_%D1%82%D0%BE%D0%BD%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D1%81%D1%82%D0%B8 "Анализ тональности")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Sentimental](https://github.com/thinkroth/Sentimental) | словарный | русский[\[2\]](https://nlpub.ru/#cite_note-2), английский | MIT | Node.js |
| [Eureka Engine](https://nlpub.ru/Eureka_Engine "Eureka Engine") | машинное обучение | русский | Коммерческая | Веб-сервис |
| [ISPRAS API Texterra](https://api.ispras.ru/) | машинное обучение | русский, английский | Бесплатная для исследовательских целей \+ коммерческая | Веб-сервис, Java, Python |
| [TextBlob](https://textblob.readthedocs.org/) | машинное обучение | английский | MIT | Python |
| [Pattern](http://www.clips.ua.ac.be/pattern) | правила, регулярные выражения | английский, испанский, немецкий, французский, итальянский, нидерландский | BSD | Python |
| [SentiStrength](http://sentistrength.wlv.ac.uk/) | словарный | русский (заявлен), английский, финский, немецкий, португальский, франузкий, арабский, польский, шведский, греческий, валлийский, итальянский | Коммерческая | Java, .NET |
| [Аналитический курьер](http://x-file.su/tm/) | правила | русский | Коммерческая | Windows |
| [DictaScope](http://www.dictum.ru/) | правила | русский | Коммерческая | FreeBSD, Windows |
| [RCO](http://rco.ru/technology.asp?ob_no=1909) | правила | русский | Коммерческая | Windows |
| [AlchemyAPI](http://www.alchemyapi.com) | машинное обучение | английский | Коммерческая | Веб-сервис |
| [Sentiment140](http://www.sentiment140.com) | машинное обучение | английский, испанский | Коммерческая | Веб-сервис |
| [ConveyAPI](https://developer.conveyapi.com) | машинное обучение | английский | Коммерческая | Веб-сервис |
| [BrandSpotter](http://brandspotter.ru/) | н/д | русский | Коммерческая | Веб-сервис |
| [RussianSentimentAnalyzer](http://semanticanalyzer.info/blog/russiansentimentanalyzer-api/) | правила | русский | Коммерческая | JSON API / Java & .NET SDK |
| [Fuxi API](http://semanticanalyzer.info/blog/chinese-sentiment-analysis-fuxi-api/) | правила | китайский | Коммерческая | JSON API |
| [NetOwl Extractor](https://www.netowl.com/entity-extraction/) | н/д | русский, английский, арабский, китайский, французский, немецкий, корейский, персидский, испанский | Коммерческая | Linux, Windows |
| [zamgi](https://github.com/zamgi/SentimentAnalysisService) | правила | русский | некоммерческая | .NET on Windows |

## [Информационный поиск](https://nlpub.ru/%D0%98%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D0%BE%D0%BD%D0%BD%D1%8B%D0%B9_%D0%BF%D0%BE%D0%B8%D1%81%D0%BA "Информационный поиск")

| Название | Метод доступа | Тип системы | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Apache Lucene Core](http://lucene.apache.org/core/) | Библиотека | полнотекстовый поиск, индекс | Apache License | Java |
| [Apache Solr](http://lucene.apache.org/solr/) | HTTP | полнотекстовый поиск, индекс | Apache License | Java |
| [AskNet Search](http://www.asknet.ru/products.htm) | HTTP | полнотекстовый поиск, [вопросно-ответный поиск](http://www.asknet.ru/), индекс | Коммерческая | Windows, С++, С# |
| [elasticsearch](http://www.elasticsearch.org) | HTTP | полнотекстовый поиск, индекс | Apache License | Java |
| [Bobo](http://senseidb.github.com/bobo/) | HTTP | фасетный поиск, индекс | Apache License | Java |
| [Picky](http://florianhanke.com/picky/) | HTTP | фасетный поиск, индекс | LGPL | Ruby |
| [Whoosh](https://bitbucket.org/mchaput/whoosh) | HTTP | полнотекстовый поиск, индекс | BSD | Python |
| [Sphinx](http://sphinxsearch.com) | Sphinx API Protocol | полнотекстовый поиск, индекс | GPL | Linux, Windows |
| [Xapian](http://xapian.org) | Библиотека | полнотекстовый поиск, индекс | GPL | Linux, Windows |
| [PostgreSQL Full Text Search](http://www.postgresql.org/docs/9.2/interactive/textsearch.html) | SQL | полнотекстовый поиск, индекс | BSD | PostgreSQL |

## [Машинный перевод](https://nlpub.ru/%D0%9C%D0%B0%D1%88%D0%B8%D0%BD%D0%BD%D1%8B%D0%B9_%D0%BF%D0%B5%D1%80%D0%B5%D0%B2%D0%BE%D0%B4 "Машинный перевод")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [Apertium](http://www.apertium.org) | конечные преобразователи, скрытые марковские модели | английский, французский, испанский, португальский, каталонский, галисийский, окситанский | GPL | Linux, Windows |
| [Moses](http://www.statmt.org/moses/) | машинное обучение | русский[\[3\]](https://nlpub.ru/#cite_note-3), английский, француский, немецкий, испанский, шведский, чешский | LGPL | Linux, Windows |
| [Sinuhe](http://www.cs.helsinki.fi/u/mtkaaria/sinuhe/) | машинное обучение | английский, немецкий, испанский, французский и другие европейские языки, для которых существует достаточного объёма параллельный корпус | (отсутствует, исходный код) | Linux |

## [Обнаружение дубликатов](https://nlpub.ru/%D0%9E%D0%B1%D0%BD%D0%B0%D1%80%D1%83%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5_%D0%B4%D1%83%D0%B1%D0%BB%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D0%B2 "Обнаружение дубликатов")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [datasketch](https://github.com/ekzhu/datasketch) | MinHash+LSH | любой | MIT | Python |

## [Cегментация текста](https://nlpub.ru/C%D0%B5%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D0%B0%D1%86%D0%B8%D1%8F_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0 "Cегментация текста")

| Название | Метод | Языки | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [zamgi](https://github.com/zamgi/lingvo--TextSegmenter) | Алгоритм Витерби | любой | MIT | .NET on Linux, Windows |

## [Интегрированные пакеты](https://nlpub.ru/%D0%98%D0%BD%D1%82%D0%B5%D0%B3%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D1%8B%D0%B5_%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D1%8B "Интегрированные пакеты")

| Название | Описание | Состав | Лицензия | Платформа |
| --- | --- | --- | --- | --- |
| [GATE](http://gate.ac.uk) | Архитектура общего назначения для обработки естественного языка | средства разработки, групповое программное обеспечение, фреймворк, общая программая архитектура, диаграммы бизнес-процессов | LGPL | Java |
| [Apache UIMA](http://uima.apache.org) | Архитектура управления неструктурированной информацией | средства разработки, фреймворк, компоненты, инфраструктура | Apache License | Java или C++ |
| [Apache OpenNLP](https://nlpub.ru/Apache_OpenNLP "Apache OpenNLP") | Инструменты обработки текста на основе машинного обучения | средства разработки, фреймворк, обученные модели | Apache License | Java или C++ |
| [SharpNLP](http://sharpnlp.codeplex.com/) | Порт [Apache OpenNLP](https://nlpub.ru/Apache_OpenNLP "Apache OpenNLP") на платформе .NET | средства разработки, фреймворк, обученные модели | LGPL | .NET |
| [NLTK](http://nltk.org) | Набор инструментов для обработки естественного языка | средства разработки, фреймворк, компоненты, инфраструктура | Apache License | Python |
| [spaCy](http://spacy.io) | Инструменты обработки текста промышленного уровня | фреймворк | MIT | Python |
| [TextBlob](https://textblob.readthedocs.org/) | Библиотека для обработки текстовых данных | фреймворк на основе [NLTK](https://nlpub.ru/index.php?title=NLTK&action=edit&redlink=1 "NLTK (страница не существует)") и Pattern | MIT | Python |
| [ISPRAS API Texterra](https://api.ispras.ru/) | Система обработки текста промышленного уровня | фреймворк, обученные модели, инфраструктура | Некоммерческая \+ коммерческая | Java, Python |
| [Treat](https://github.com/louismullie/treat) | Набор утилит для обработки естественного языка и компьютерной лингвистики на языке Ruby | средства разработки, фреймворк, слой интеграции со сторонними продуктами, обученные модели | GPL | Ruby |
| [Linguistics](https://github.com/ged/linguistics) | Языконезависимый фреймворк для расширения Ruby-объектов методами обработки текста общего назначения | языконезависимая оболочка, отображение кодов языка в названия, утилиты | MIT? | Ruby |
| [NooJ](http://www.nooj4nlp.net/pages/nooj.html) | Среда разработки лингвистических инструментов | словари, грамматики, анализаторы, таггеры | AGPL | Java, .NET |
| [Stanford NLP](http://nlp.stanford.edu/software/index.shtml) | Программное обеспечение для обработки естественного языка, доступное каждому | фреймворк | GPL + Коммерческая | Java |
| [MinorThird](http://teamcohen.github.io/MinorThird/) | Набор Java-классов для обработки естественного языка | решения для хранения и разметки текста, средства для машинного обучения | BSD | Java |
| [Grammatical Framework](http://www.grammaticalframework.org/) | Язык программирования для обработки естественного языка | средства разработки, фреймворк, компоненты, инфраструктура | GPL (программа) и LGPL и BSD (библиотеки) | н/д |
| [libschwa](https://github.com/schwa-lab/libschwa) | Инструменты для обработки текстов от Schwa Lab | фреймворк | MIT | C++ |
| [natural](https://github.com/NaturalNode/natural) | Общие средства обработки естественного языка для Node.js | анализаторы | MIT | Node.js |
| [LingPipe](http://alias-i.com/lingpipe/) | Пакет инструментов для обработки текста средствами компьютерной лингвистики | фреймворк, обученные модели, средства для многопоточной работы, тесты | Коммерческая и некоммерческая | Java |
| [T-LAB](http://www.tlab.it/en/presentation.php) | Инструменты для анализа текста | инструменты для анализа тематики, сравнительного анализа, анализа совместной встречаемости | Коммерческая | н/д |
| [MeTA](https://meta-toolkit.github.io/meta/) | Современный набор утилит на C++ для науки о данных | фреймворк | MIT | C++ |
| [Eureka Engine](https://nlpub.ru/Eureka_Engine "Eureka Engine") | Набор инструментов для обработк�� естественного языка | средства разработки, фреймворк, компоненты, инфраструктура | Коммерческая | н/д |
| [zamgi](https://github.com/zamgi) | Набор инструментов для обработки естественного языка | средства разработки, фреймворк, компоненты, инфраструктура | некоммерческая | .NET on Linux, Windows |

## Примечания

1.  [Перейти ↑](https://nlpub.ru/#cite_ref-1) Существует проект [The AGFL for the Russian Language](http://www.agfl.cs.ru.nl/rus/index.html).
2.  [Перейти ↑](https://nlpub.ru/#cite_ref-2) Для Sentimental имеется [поддержка русского языка](https://github.com/Wobot/Sentimental), реализованная [компанией](https://nlpub.ru/%D0%9E%D1%80%D0%B3%D0%B0%D0%BD%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8 "Организации") Wobot.
3.  [Перейти ↑](https://nlpub.ru/#cite_ref-3) Известна адаптация системы Moses для русского языка в виде [демонстрационного Веб-сервиса](http://sz.ru/smt/).

Источник — «[https://nlpub.ru/index.php?title=Обработка_текста&oldid=2777](https://nlpub.ru/index.php?title=Обработка_текста&oldid=2777)»

[Категория](https://nlpub.ru/%D0%A1%D0%BB%D1%83%D0%B6%D0%B5%D0%B1%D0%BD%D0%B0%D1%8F:%D0%9A%D0%B0%D1%82%D0%B5%D0%B3%D0%BE%D1%80%D0%B8%D0%B8 "Служебная:Категории"):

*   [Инструменты](https://nlpub.ru/%D0%9A%D0%B0%D1%82%D0%B5%D0%B3%D0%BE%D1%80%D0%B8%D1%8F:%D0%98%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82%D1%8B "Категория:Инструменты")

## Навигация

### Персональные инструменты

*   [Создать учётную запись](https://nlpub.ru/index.php?title=%D0%A1%D0%BB%D1%83%D0%B6%D0%B5%D0%B1%D0%BD%D0%B0%D1%8F:%D0%A1%D0%BE%D0%B7%D0%B4%D0%B0%D1%82%D1%8C_%D1%83%D1%87%D1%91%D1%82%D0%BD%D1%83%D1%8E_%D0%B7%D0%B0%D0%BF%D0%B8%D1%81%D1%8C&returnto=%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0+%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0 "Мы предлагаем вам создать учётную запись и войти в систему, хотя это и не обязательно.")
*   [Войти](https://nlpub.ru/index.php?title=%D0%A1%D0%BB%D1%83%D0%B6%D0%B5%D0%B1%D0%BD%D0%B0%D1%8F:%D0%92%D1%85%D0%BE%D0%B4&returnto=%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0+%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0 "Здесь можно зарегистрироваться в системе, но это необязательно. [alt-shift-o]")

### Пространства имён

*   [Статья](https://nlpub.ru/%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0 "Просмотр основной страницы [alt-shift-c]")
*   [Обсуждение](https://nlpub.ru/index.php?title=%D0%9E%D0%B1%D1%81%D1%83%D0%B6%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5:%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0&action=edit&redlink=1 "Обсуждение основной страницы (страница не существует) [alt-shift-t]")

### Варианты

### Просмотры

*   [Читать](https://nlpub.ru/%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0)
*   [Просмотр кода](https://nlpub.ru/index.php?title=%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0&action=edit "Эта страница защищена от изменений, но вы можете посмотреть и скопировать её исходный текст [alt-shift-e]")
*   [История](https://nlpub.ru/index.php?title=%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0&action=history "Журнал изменений страницы [alt-shift-h]")

### Ещё

### Поиск

### NLPub

*   [Заглавная страница](https://nlpub.ru/%D0%97%D0%B0%D0%B3%D0%BB%D0%B0%D0%B2%D0%BD%D0%B0%D1%8F_%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B8%D1%86%D0%B0 "Перейти на заглавную страницу [alt-shift-z]")
*   [Обработка текста](https://nlpub.ru/%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0)
*   [Обработка речи](https://nlpub.ru/%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%80%D0%B5%D1%87%D0%B8)
*   [Утилиты](https://nlpub.ru/%D0%A3%D1%82%D0%B8%D0%BB%D0%B8%D1%82%D1%8B)
*   [Методы](https://nlpub.ru/%D0%9C%D0%B5%D1%82%D0%BE%D0%B4%D1%8B)
*   [Алгоритмы](https://nlpub.ru/%D0%90%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC%D1%8B)

### Ресурсы

*   [Словари](https://nlpub.ru/%D0%A1%D0%BB%D0%BE%D0%B2%D0%B0%D1%80%D1%8C)
*   [Тезаурусы](https://nlpub.ru/%D0%A2%D0%B5%D0%B7%D0%B0%D1%83%D1%80%D1%83%D1%81)
*   [Корпусы](https://nlpub.ru/%D0%9A%D0%BE%D1%80%D0%BF%D1%83%D1%81_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%BE%D0%B2)
*   [Банки данных](https://nlpub.ru/%D0%91%D0%B0%D0%BD%D0%BA_%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85)
*   [Журналы](https://nlpub.ru/%D0%96%D1%83%D1%80%D0%BD%D0%B0%D0%BB%D1%8B)

### Вживую

*   [Мероприятия](https://nlpub.ru/%D0%9C%D0%B5%D1%80%D0%BE%D0%BF%D1%80%D0%B8%D1%8F%D1%82%D0%B8%D1%8F)
*   [Персоналии](https://nlpub.ru/%D0%9F%D0%B5%D1%80%D1%81%D0%BE%D0%BD%D0%B0%D0%BB%D0%B8%D0%B8)
*   [Организации](https://nlpub.ru/%D0%9E%D1%80%D0%B3%D0%B0%D0%BD%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8)
*   [Образование](https://nlpub.ru/%D0%9E%D0%B1%D1%80%D0%B0%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)
*   [Темы дипломов](https://nlpub.ru/%D0%A2%D0%B5%D0%BC%D1%8B_%D0%B4%D0%B8%D0%BF%D0%BB%D0%BE%D0%BC%D0%BE%D0%B2)
*   [Литература](https://nlpub.ru/%D0%9B%D0%B8%D1%82%D0%B5%D1%80%D0%B0%D1%82%D1%83%D1%80%D0%B0)

### Навигация

*   [Свежие правки](https://nlpub.ru/%D0%A1%D0%BB%D1%83%D0%B6%D0%B5%D0%B1%D0%BD%D0%B0%D1%8F:%D0%A1%D0%B2%D0%B5%D0%B6%D0%B8%D0%B5_%D0%BF%D1%80%D0%B0%D0%B2%D0%BA%D0%B8 "Список последних изменений [alt-shift-r]")
*   [Случайная статья](https://nlpub.ru/%D0%A1%D0%BB%D1%83%D0%B6%D0%B5%D0%B1%D0%BD%D0%B0%D1%8F:%D0%A1%D0%BB%D1%83%D1%87%D0%B0%D0%B9%D0%BD%D0%B0%D1%8F_%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B8%D1%86%D0%B0 "Посмотреть случайно выбранную страницу [alt-shift-x]")
*   [Справка](https://www.mediawiki.org/wiki/Special:MyLanguage/Help:Contents "Место, где можно получить справку")

### Инструменты

*   [Ссылки сюда](https://nlpub.ru/%D0%A1%D0%BB%D1%83%D0%B6%D0%B5%D0%B1%D0%BD%D0%B0%D1%8F:%D0%A1%D1%81%D1%8B%D0%BB%D0%BA%D0%B8_%D1%81%D1%8E%D0%B4%D0%B0/%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0 "Список всех страниц, ссылающихся на данную [alt-shift-j]")
*   [Связанные правки](https://nlpub.ru/%D0%A1%D0%BB%D1%83%D0%B6%D0%B5%D0%B1%D0%BD%D0%B0%D1%8F:%D0%A1%D0%B2%D1%8F%D0%B7%D0%B0%D0%BD%D0%BD%D1%8B%D0%B5_%D0%BF%D1%80%D0%B0%D0%B2%D0%BA%D0%B8/%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0 "Последние изменения в страницах, на которые ссылается эта страница [alt-shift-k]")
*   [Спецстраницы](https://nlpub.ru/%D0%A1%D0%BB%D1%83%D0%B6%D0%B5%D0%B1%D0%BD%D0%B0%D1%8F:%D0%A1%D0%BF%D0%B5%D1%86%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B8%D1%86%D1%8B "Список служебных страниц [alt-shift-q]")
*   [Версия для печати](https://nlpub.ru/index.php?title=%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0&printable=yes "Версия этой страницы для печати [alt-shift-p]")
*   [Постоянная ссылка](https://nlpub.ru/index.php?title=%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0&oldid=2777 "Постоянная ссылка на эту версию страницы")
*   [Сведения о странице](https://nlpub.ru/index.php?title=%D0%9E%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0_%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0&action=info "Подробнее об этой странице")

*   Эта страница последний раз была отредактирована 24 июля 2018 в 12:58.
*   Содержание доступно по лицензии [Creative Commons Attribution ShareAlike](https://nlpub.ru/NLPub:%D0%9B%D0%B8%D1%86%D0%B5%D0%BD%D0%B7%D0%B8%D1%8F "NLPub:Лицензия") (если не указано иное).

*   [Политика конфиденциальности](https://nlpub.ru/NLPub:%D0%9F%D0%BE%D0%BB%D0%B8%D1%82%D0%B8%D0%BA%D0%B0_%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B4%D0%B5%D0%BD%D1%86%D0%B8%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D1%81%D1%82%D0%B8 "NLPub:Политика конфиденциальности")
*   [О NLPub](https://nlpub.ru/NLPub:%D0%9E%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5 "NLPub:Описание")
*   [Отказ от ответственности](https://nlpub.ru/NLPub:%D0%9E%D1%82%D0%BA%D0%B0%D0%B7_%D0%BE%D1%82_%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D1%81%D1%82%D0%B2%D0%B5%D0%BD%D0%BD%D0%BE%D1%81%D1%82%D0%B8 "NLPub:Отказ от ответственности")

*   [![Creative Commons Attribution ShareAlike](../_resources/3109617551744fefb1708801a31c490e.png)](http://creativecommons.org/licenses/by-sa/4.0/)
*   [![Powered by MediaWiki](../_resources/f85c5ea9ec4143c291fbbe10b32c30b6.png)](https://www.mediawiki.org/)