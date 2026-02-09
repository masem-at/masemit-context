Gute Frage — und laut unserem Leitfaden ist die Antwort klarer als     man denkt:                                                           
                                                                         Was KEIN Projektstart ist (= sicher VOR Antragstellung):
  ┌───────────────────────────────────────┬───────────────────┐          │               Aktivität               │      Status       │        
  ├───────────────────────────────────────┼───────────────────┤
  │ Research & Market Analysis            │ Bereits erledigt  │        
  ├───────────────────────────────────────┼───────────────────┤        
  │ Product Brief                         │ Gerade fertig     │        
  ├───────────────────────────────────────┼───────────────────┤        
  │ Product Requirements Document (PRD)   │ Können wir machen │        
  ├───────────────────────────────────────┼───────────────────┤        
  │ UX-Design, Wireframes, Mockups        │ Können wir machen │        
  ├───────────────────────────────────────┼───────────────────┤        
  │ Architektur-Planung & Tech-Evaluation │ Können wir machen │        
  ├───────────────────────────────────────┼───────────────────┤        
  │ Landing Page + Warteliste             │ Können wir machen │        
  ├───────────────────────────────────────┼───────────────────┤        
  │ Gespräche mit Pilotbetrieben          │ Können wir machen │        
  ├───────────────────────────────────────┼───────────────────┤        
  │ FFG-Antrag schreiben & einreichen     │ Müssen wir machen │        
  └───────────────────────────────────────┴───────────────────┘        
  Was Projektstart IST (= ERST nach Bewilligung):
  ┌──────────────────────────────────────────┬────────┐
  │                Aktivität                 │ Status │
  ├──────────────────────────────────────────┼────────┤
  │ MVP-Programmierung                       │ STOP   │
  ├──────────────────────────────────────────┼────────┤
  │ Lizenzen/Services kaufen für das Projekt │ STOP   │
  ├──────────────────────────────────────────┼────────┤
  │ Externe Dienstleister beauftragen        │ STOP   │
  └──────────────────────────────────────────┴────────┘
  Kurzum: Wir können den gesamten BMAD-Planungszyklus durchlaufen —    
  PRD, UX-Design, Architektur, alles. Wir dürfen nur nicht anfangen zu 
  coden.

  Das ist sogar ideal: Wenn der FFG-Antrag bewilligt wird, haben wir   
  alles fix und fertig geplant und können sofort loslegen. Der Antrag  
  wird auch stärker, wenn wir eine durchdachte Planung vorlegen können.

  Grauzone laut unserem Leitfaden: Ein "Low-Cost-Prototyp" zur reinen  
  Technologie-Evaluation (z.B. Mistral API testen mit deinem
  bestehenden Vercel/NeonDB-Konto) könnte als Research durchgehen —    
  aber im Zweifelsfall vorher bei der FFG anrufen.

  Sollen wir mit dem PRD weitermachen?

✻ Brewed for 46s                                                         
❯ Lizenzen/Services kaufen für das Projekt                             

  vercel, neondb, verwende ich ja schon für andere projekte, das         darf ich halt dann nicht verrechnen oder wie?                        
                                                                       ● Genau richtig verstanden. Deine bestehende Infrastruktur (Vercel Pro,   NeonDB Launch, Upstash, Resend) läuft ja schon für TellingCube,     
  ChainSights etc. — das ist kein neuer Kauf für kurzum.app.              
  Die Unterscheidung ist:                                              
  ┌──────────────────────────────┬────────────────┬───────────────────┐
  │          Situation           │    Erlaubt?    │  Im FFG-Antrag    │
  │                              │                │   verrechenbar?   │
  ├──────────────────────────────┼────────────────┼───────────────────┤
  │ Bestehende Services weiter   │ Ja, kein       │ Nein (existierten │
  │ nutzen (Vercel, NeonDB...)   │ Problem        │  vorher)          │
  ├──────────────────────────────┼────────────────┼───────────────────┤
  │ Auf bestehendem Konto was    │ Ja, ist        │ Nein              │
  │ für kurzum testen/planen     │ Research       │                   │
  ├──────────────────────────────┼────────────────┼───────────────────┤
  │ Services UPGRADEN wegen      │ Erst nach      │ Ja, ab            │
  │ kurzum (z.B. höherer         │ Bewilligung    │ Projektstart      │
  │ NeonDB-Tier)                 │                │                   │
  ├──────────────────────────────┼────────────────┼───────────────────┤
  │ NEUE Services kaufen für     │ Erst nach      │ Ja, ab            │
  │ kurzum (z.B. Mistral         │ Bewilligung    │ Projektstart      │
  │ API-Plan)                    │                │                   │
  └──────────────────────────────┴────────────────┴───────────────────┘
  Das "keine Lizenzen kaufen" bezieht sich auf neue, projektspezifische
   Kosten die du als Förderkosten geltend machen willst. Dein
  bestehendes Vercel-Konto zu nutzen um eine Landing Page zu bauen oder
   eine Architektur zu skizzieren ist völlig unbedenklich — das zahlst 
  du sowieso schon.

  Tipp für den FFG-Antrag: Du kannst dort die anteiligen Kosten der    
  bestehenden Infrastruktur ab Projektstart als "Infrastrukturkosten"  
  ansetzen — aber erst ab dem bewilligten Startdatum. Am besten bei der
   FFG nachfragen wie sie anteilige Nutzung bestehender Services       
  handhaben.