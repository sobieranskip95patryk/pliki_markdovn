# ActivationEngine — MIGI_7G

## Cel
Silnik odpowiedzialny za wykonywanie reguł aktywacyjnych i generowanie MIGI.Context.

## Wejście
- symbol (np. SEED-01)
- idea
- sequence
- application

## Wyjście
- MIGI.Context zgodny z context.schema.json

## Proces
1. Odczytaj symbol modułu LOGOS.
2. Znajdź powiązaną regułę (np. RULE-0001-SEED-ACTIVATION).
3. Wykonaj kroki reguły.
4. Wygeneruj MIGI.Context.
5. Zwróć wynik do GlobalVision.

## Powiązania
- LOGOS: Module_0001_Seed
- MIGI_7G: RULE-0001-SEED-ACTIVATION
- MIGI_7G: context.schema.json
- GlobalVision: Model_0001_System_Trajectory
