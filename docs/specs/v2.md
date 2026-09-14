# Reitschule – Domänenmodell, Entwurf v2

Dieser Entwurf beschreibt den fachlichen Kern des ersten Inkrements. Es digitalisiert die Erfassung von Unterricht und
die monatliche Abrechnung der Trainer. Schülerabrechnung, Vertragsverwaltung und die automatische Prüfung von
Nachholberechtigungen folgen bewusst später.

Die Anwendung ist bewusst eine eigenständige Insellösung. Schnittstellen oder ein Datenaustausch mit Drittsystemen sind
nicht vorgesehen.

## Domänenentitäten

- **Schule**: Der Verein beziehungsweise die Reitschule, in deren Kontext alle Daten geführt werden. Eine Installation
  bedient genau eine Schule.
- **Admin**: Vertrauenswürdiger Benutzer der Admin-Anwendung. Admins pflegen Stammdaten und dürfen Unterricht sowie
  Trainerabrechnungen auch nach deren Einreichung korrigieren. Inaktive Admins haben keinen Zugriff.
- **Trainer**: Benutzer der Trainer-Anwendung. Trainer erfassen ihren eigenen Unterricht und reichen ihre monatliche
  Abrechnung ein. Inaktive Trainer haben keinen Zugriff.
- **Schüler**: Person, die an Unterrichtseinheiten teilnimmt. Im ersten Inkrement wird nur der dafür erforderliche
  Stammdatensatz geführt. Nur aktive Schüler können für neue Teilnahmen ausgewählt werden.
- **Pferd**: Schulpferd, das einer Teilnahme zugeordnet werden kann. Die Zuordnung ist optional, da Unterricht auch ohne
  Pferd stattfinden kann. Nur aktive Pferde können für neue Teilnahmen ausgewählt werden.
- **Unterrichtsart**: Fachliche Art des Unterrichts, beispielsweise Gruppe, Einzel oder Longe. Sie stellt übliche Dauer
  und Trainervergütung als Vorgabe bereit. Für Sonderfälle kann eine individuelle Unterrichtsart mit freier Bezeichnung
  und Vergütung erfasst werden.
- **Unterricht**: Ein von einem Trainer zu einem bestimmten Zeitpunkt durchgeführter Unterricht. Er besteht aus einer
  oder mehreren klar abgegrenzten Unterrichtseinheiten.
- **Unterrichtseinheit**: Eine einzelne erbrachte und abrechenbare Einheit innerhalb eines Unterrichts. Eine
  Doppelstunde besteht beispielsweise aus zwei Unterrichtseinheiten. Die Dauer kann je nach Unterrichtsart 30, 45 oder
  60 Minuten betragen; es gibt keine universelle 45-Minuten-Einheit.
- **Teilnahme**: Die Teilnahme eines Schülers an einer bestimmten Unterrichtseinheit. Sie enthält optional das gerittene
  Pferd und kann als Nachholteilnahme markiert werden. Bei einer Nachholteilnahme kann zusätzlich das Datum der
  versäumten Einheit erfasst werden. Ein Schüler kann an mehreren Einheiten desselben Unterrichts teilnehmen und dabei
  unterschiedliche Pferde reiten.
- **Trainerabrechnung**: Abrechnung eines Trainers für einen Kalendermonat. Sie ist zunächst ein Entwurf und wird durch
  den Trainer eingereicht. Danach sind die enthaltenen Daten für diesen Trainer gesperrt; Admins dürfen sie weiterhin
  korrigieren. Andere Trainer bleiben davon unberührt.
- **Zusätzliche Abrechnungsposition**: Freie Position einer Trainerabrechnung, die nicht aus Unterricht entsteht, etwa
  eine Auslage für eine neue Trense. Sie besteht mindestens aus Beschreibung und Betrag.

## Fachliche Zusammenhänge

Die Trainervergütung entsteht pro erbrachter Unterrichtseinheit, nicht pro teilnehmendem Schüler. Eine doppelte
Gruppenstunde umfasst somit zwei vergütete Gruppeneinheiten. Die Teilnahmen dokumentieren unabhängig davon, welche
Schüler in welcher Einheit mit welchem Pferd geritten sind.

Beispiel: Ein Schüler reitet in der ersten Einheit einer Doppelstunde regulär auf Pferd A. In der zweiten Einheit reitet
er auf Pferd B und holt eine zuvor versäumte Einheit nach. Das sind zwei Teilnahmen an zwei Unterrichtseinheiten
desselben Unterrichts.

Die Unterrichtsart, der vom Schüler zu zahlende Tarif und die Trainervergütung sind unterschiedliche fachliche
Konzepte. Das erste Inkrement benötigt Unterrichtsarten und Trainervergütungen. Schülertarife und Schülerforderungen
werden noch nicht verwaltet.

Auswertungen über Unterricht, Trainer, Schüler oder Pferde sind abgeleitete Ansichten und keine eigenen
Domänenentitäten. Die Trainerabrechnung ist dagegen dauerhaft gespeichert, weil ihre Einreichung den Bearbeitungsstatus
verändert.

## Entscheidungen für das erste Inkrement

- Im Mittelpunkt steht der vollständige Ablauf von Unterrichtserfassung bis zur eingereichten Trainerabrechnung.
- Eine globale Monatsschließung gibt es nicht. Jeder Trainer reicht seinen Monat unabhängig ein.
- Der 6. des Folgemonats ist eine organisatorische Frist, aber keine technische Sperrfrist.
- Pferde sind pro Teilnahme optional; aktive Schüler und Pferde werden ausgewählt und nicht frei eingetippt.
- Nachholstunden werden zunächst nur gekennzeichnet und optional mit dem Datum der versäumten Einheit versehen.
- Die Einhaltung von Anmeldefrist, Papiernachweis und Nachholfrist wird noch nicht automatisch geprüft.
- Ein allgemeines Quota-Konto mit austauschbaren 45-Minuten-Einheiten wird nicht eingeführt.
- Schülerverträge und Schülertarife folgen später.
- Historische Stammdaten bleiben erhalten; nicht mehr benötigte personenbezogene Daten werden anonymisiert statt
  fachliche Historie zu löschen.

## Beantwortete Fragen

- **Kann Unterricht ohne Pferd stattfinden?** Ja; deshalb ist die Pferdezuordnung optional.
- **Kann ein Schüler innerhalb eines Unterrichts mehrfach teilnehmen?** Ja; jeweils an einer konkreten
  Unterrichtseinheit, beispielsweise bei einer Doppelstunde.
- **Kann dabei das Pferd wechseln?** Ja; jede Teilnahme besitzt ihre eigene optionale Pferdezuordnung.
- **Wird der Trainer pro Schüler bezahlt?** Nein; die Vergütung richtet sich nach den erbrachten Unterrichtseinheiten.
- **Dürfen Trainer andere abrechenbare Positionen erfassen?** Ja; als zusätzliche Freitextposition mit Betrag.
- **Wann werden Trainerdaten gesperrt?** Bei der individuellen Einreichung der Monatsabrechnung, nicht zu einem festen
  Datum und nicht durch einen globalen Monatsabschluss.
- **Dürfen Admins danach korrigieren?** Ja.
- **Sind alle Unterrichtseinheiten gleich lang?** Nein; ihre Dauer hängt von der Unterrichtsart und dem konkreten
  Unterricht ab.

## Noch offene Fragen für spätere Inkremente

- Bezeichnet ein gewünschter individueller „Blanko-Preis“ den Schülerpreis, die Trainervergütung oder beides?
- Gelten Trainervergütungen einheitlich oder unterscheiden sie sich nach Trainer und Gültigkeitszeitraum?
- Kann eine eingereichte Trainerabrechnung zur erneuten Bearbeitung zurückgegeben werden?
- Welche Schülervereinbarungen dürfen gleichzeitig bestehen, und wie werden daraus Ansprüche auf Unterricht abgeleitet?
- Wie werden rechtzeitig gemeldete Abwesenheiten, Nachholberechtigungen, deren Ablauf und vom Verein verursachte Ausfälle
  voneinander unterschieden?
