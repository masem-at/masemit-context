# ADR-001: Mistral AI Scale Plan statt Experiment

**Datum:** 2026-02-13
**Status:** Entschieden
**Kontext:** Mistral Experiment Plan erlaubt Training mit API-Daten. 
             Wir verarbeiten Sprachdaten von Handwerkern → personenbezogene Daten.
**Entscheidung:** Scale Plan (Pay-per-use) für Produktion/Pilot.
                  Experiment Plan nur für synthetische Testdaten.
**Begründung:** DSGVO Art. 6 – keine Rechtsgrundlage für Training durch Dritte.
                FFG-Antrag sagt "DSGVO-by-Design".