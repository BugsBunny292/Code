#include <LedControl.h>

// LedControl: DIN, CLK, CS, number of displays
LedControl lc = LedControl(11, 13, 10, 1);

int birdPosition = 4; // Kuşun başlangıç pozisyonu (y ekseni)
int pipeX = 7;        // Boruların başlangıç pozisyonu (x ekseni)
int pipeGap = 3;      // Borudaki boşluğun konumu (y ekseni)
int joystickYPin = A1; // Joystick Y ekseni
int score = 0;        // Skor

unsigned long previousMillis = 0;
unsigned long birdMoveMillis = 0;  // Kuşun hareket zamanlayıcısı
const long interval = 200;  // Boruların hareket etme süresi (ms)
const long birdMoveInterval = 150;  // Kuşun hareket aralığı (ms)

void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 8); // Parlaklık ayarı
  lc.clearDisplay(0);
  Serial.begin(9600);
}

void loop() {
  unsigned long currentMillis = millis();

  // Kuşun hareket hızını kontrol etmek için bir zamanlayıcı ekledik
  if (currentMillis - birdMoveMillis >= birdMoveInterval) {
    birdMoveMillis = currentMillis;

    // Joystick Y eksenini oku
    int joystickY = analogRead(joystickYPin);
  
    // Joystick yukarı hareketi (kuş yukarı çıkıyor)
    if (joystickY < 400) {
      birdPosition = max(0, birdPosition - 1); // Yukarı hareket
    }
    // Joystick aşağı hareketi (kuş aşağı iniyor)
    else if (joystickY > 600) {
      birdPosition = min(7, birdPosition + 1); // Aşağı hareket
    }
  }

  // Boruların hareketi
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    pipeX--;
    if (pipeX < 0) {
      pipeX = 7;
      pipeGap = random(1, 6);  // Yeni boru boşluğu rastgele ayarlanır
      score++;  // Her borudan geçişte skor artırılır
      Serial.print("Score: ");
      Serial.println(score);  // Skor seri monitöre yazdırılır
    }
  }

  // Çarpma kontrolü
  if (pipeX == 0 && (birdPosition < pipeGap || birdPosition > pipeGap + 1)) {
    gameOver();
  }

  // Ekranı temizle ve kuş ile boruları çiz
  lc.clearDisplay(0);
  lc.setLed(0, birdPosition, 0, true);  // Kuşun konumu
  drawPipes();
}

void drawPipes() {
  for (int y = 0; y < 8; y++) {
    if (y < pipeGap || y > pipeGap + 1) {
      lc.setLed(0, y, pipeX, true);  // Boruyu çiz
    }
  }
}

void gameOver() {
  lc.clearDisplay(0);
  for (int i = 0; i < 8; i++) {
    lc.setLed(0, i, i, true);
    delay(100);
  }
  delay(1000);
  
  // Skoru göster
  Serial.print("Game Over! Final Score: ");
  Serial.println(score);

  // Başlangıç değerlerine dön
  birdPosition = 4;
  pipeX = 7;
  pipeGap = 3;
  score = 0;  // Skoru sıfırla
}
