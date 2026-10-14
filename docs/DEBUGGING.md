# Felsökningsrapport

## Vilket fel undersökte du?
Jag undersökte felet med överlappande bokningar. Problemet var att applikationen tillät en användare att göra en ny bokning 
även om mötesrummet var redan bokat under samma tid. Detta var självklart ett problem eftersom två kunder kunde boka samma rum samtidfigt. 

## Hur återskapade du felet?
Jag började med att kolla igenom testerna som fanns och såg att det fanns ett test "Create_WhenBookingOverlapsExistingBooking_ReturnsFailure"
som testade just detta scenario, flera olika tider mot en befintlig bokning mellan 09:00 och 11:00. 

## Vilket resultat fick du?
När jag körde testerna såg jag att två av de fyra överlappningsfallen misslyckades.

## Vilket resultat skulle du ha fått?
Det förväntade resultatet var att alla fyra överlappande bokningar skulle nekas med ett felmeddelande som säger att rummet redan är bokat under
den tiden. Däremot skulle en bokning som började exakt när den befintliga bokningen slutade godkännas, eftersom tiderna då inte överlappar.

## Vilken del av koden orsakade felet?
Jag hittade felet i BookingService i metoden Validate. Den gamla koden var detta: 

booking.StartTime >= existing.StartTime &&
booking.StartTime < existing.EndTime

## Varför gav koden ett felaktigt resultat?
Problemet med denna kod var att koden kontrollerade bara om den nya bokningens starttid låg inom den befintliga bokningesn tidsintervall. Om den 
befintliga bokningen var mellan 09:00-11:00 och den nya bokningen var 08:00-10:00 då tyckte applikationen att det var okej och att det inte fanns någon överlappning.
Detta berodde på att 08:00 inte är större än eller lika med 09:00, vilket gjorde att det första villkoret blev false och programmet missade överlappningen.

## Vad ändrade du?
Jag ändrade tidskontrollen till:

booking.StartTime < existing.EndTime &&
booking.EndTime > existing.StartTime

Nu kontrollerar programmet både att den nya bokningen börjar innan den befintliga slutar och att den nya bokningen slutar efter att den befintliga börjar.

## Hur kontrollerade du att ändringen fungerade?
Efter ändringen körde jag de automatiserade testerna igen. Då gick alla fyra fallen i Create_WhenBookingOverlapsExistingBooking_ReturnsFailure igenom.
Jag kontrollerade också att Create_WhenBookingStartsWhenExistingBookingEnds_ReturnsSuccess fortfarande fungerade.Sedan testade jag manuellt i konsolen
genom att försöka boka Eken mellan 14:30 och 15:30 när rummet redan var bokat mellan 15:00 och 16:00. Programmet nekade då bokningen som förväntat.

## Vilket automatiserat test skyddar mot att felet återkommer?
Det automatiserade testet som skyddar mot att felet återkommer är Create_WhenBookingOverlapsExistingBooking_ReturnsFailure i BookingServiceTests.cs. 
Testet kontrollerar fyra olika situationer där en ny bokning överlappar en befintlig bokning. Jag kontrollerade också testet 
Create_WhenBookingStartsWhenExistingBookingEnds_ReturnsSuccess, som ser till att en ny bokning fortfarande kan godkännas om den börjar exakt när 
den tidigare bokningen slutar.

## Ett annat alternativ
En annan lösning hade kunnat vara att använda flera olika if-satser för att kontrollera olika situationer där bokningarna kan överlappa. Men jag 
valde att använda två villkor tillsammans med && eftersom det är enklare och kräver mindre kod. Det gör också att vi kan kontrollera olika typer av 
överlappningar på samma gång.

