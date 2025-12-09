
import 'package:flutter/material.dart';

  

// --- Modelos ---

  

class BodyPart {

 final String id;

 final String name;

 double percentage;

 // Coordenadas para centralizar o texto no desenho

 final double textX;

 final double textY;

  

 BodyPart({

  required this.id,

  required this.name,

  required this.percentage,

  required this.textX,

  required this.textY,

 });

}

  

// --- Custom Painter para desenhar o corpo ---

  

class BodyPainter extends CustomPainter {

 final List<BodyPart> parts;

 final String? hoveredPartId;

  

 BodyPainter(this.parts, this.hoveredPartId);

  

 // Lógica de cores

 Color getColor(double pct) {

  if (pct < 30) return Colors.red.shade500;

  if (pct < 60) return Colors.yellow.shade400;

  return Colors.green.shade500;

 }

  

 // Lógica para desenhar o brilho (glow effect)

 List<Shadow> getGlow(double pct) {

  Color glowColor;

  if (pct < 30) {

   glowColor = Colors.red.shade500.withOpacity(0.6);

  } else if (pct < 60) {

   glowColor = Colors.yellow.shade400.withOpacity(0.5);

  } else {

   glowColor = Colors.green.shade500.withOpacity(0.6);

  }

  

  return [

   Shadow(

    color: glowColor,

    blurRadius: 15.0,

    offset: Offset.zero,

   ),

  ];

 }

  

 // Função para desenhar o polígono ou círculo

 void drawPart({

  required Canvas canvas,

  required Path path,

  required BodyPart part,

  bool isCircle = false,

 }) {

  final color = getColor(part.percentage);

  final glowShadows = getGlow(part.percentage);

  

  final paint = Paint()

   ..color = color

   ..style = PaintingStyle.fill;

  

  if (isCircle) {

   // Cabeça é um círculo

   canvas.drawCircle(Offset(part.textX, part.textY - 5), 50, paint);

  } else {

   // Outras partes são Paths

   canvas.drawPath(path, paint);

  }

  

  // Desenha o texto da porcentagem

  final textPainter = TextPainter(

   text: TextSpan(

    text: '${part.percentage.toInt()}%',

    style: TextStyle(

     color: Colors.black.withOpacity(0.6),

     fontWeight: FontWeight.bold,

     fontSize: part.id == 'torso' ? 22 : 16,

     shadows: glowShadows, // Adicionando o glow aqui

    ),

   ),

   textDirection: TextDirection.ltr,

  );

  

  textPainter.layout();

  textPainter.paint(

   canvas,

   Offset(part.textX - textPainter.width / 2, part.textY - textPainter.height / 2),

  );

  

  // Desenha um destaque se estiver sob hover

  if (hoveredPartId == part.id) {

   final highlightPaint = Paint()

    ..color = Colors.white.withOpacity(0.5)

    ..style = PaintingStyle.stroke

    ..strokeWidth = 3.0;

   if (isCircle) {

    canvas.drawCircle(Offset(part.textX, part.textY - 5), 52, highlightPaint);

   } else {

    canvas.drawPath(path, highlightPaint);

   }

  }

 }

  

 @override

 void paint(Canvas canvas, Size size) {

  // Escala para garantir que o desenho se ajuste à caixa (se ViewBox for 400x600)

  final scaleFactor = size.width / 400.0;

  canvas.scale(scaleFactor);

  // Coordenadas-chave para conexões suaves (base 400x600)

  const double shoulderY = 130;

  const double hipY = 290;

  const double crotchY = 310;

  const double upperArmXLeft = 150;

  const double upperArmXRight = 250;

  const double hipInnerXLeft = 170;

  const double hipInnerXRight = 230;

  // Linha de conexão Cabeça/Tronco (mais suave)

  final connectorPaint = Paint()

   ..color = const Color(0xFF334155) // slate-700

   ..strokeWidth = 8.0

   ..style = PaintingStyle.stroke

   ..strokeCap = StrokeCap.round;

  canvas.drawLine(const Offset(200, 120), const Offset(200, 130), connectorPaint);

  

  // --- 2. TRONCO / PEITO (Conexões redefinidas) ---

  final torso = parts.firstWhere((p) => p.id == 'torso');

  final torsoPath = Path()

   ..moveTo(upperArmXLeft, shoulderY) // Upper Left Shoulder Point (Conecta com l_arm)

   ..quadraticBezierTo(200, 100, upperArmXRight, shoulderY) // Upper Chest Curve (Neck area)

   ..lineTo(upperArmXRight, hipY) // Right side down

   ..quadraticBezierTo(hipInnerXRight, crotchY, 200, crotchY) // Lower right hip to center

   ..quadraticBezierTo(hipInnerXLeft, crotchY, upperArmXLeft, hipY) // Lower center to left hip

   ..close();

  drawPart(canvas: canvas, path: torsoPath, part: torso);

  

  // --- 3. BRAÇO ESQUERDO (Conectado ao ombro) ---

  final lArm = parts.firstWhere((p) => p.id == 'l_arm');

  final lArmPath = Path()

   // Começa no ponto do ombro do torso (upperArmXLeft, shoulderY)

   ..moveTo(upperArmXLeft, shoulderY)

   ..lineTo(upperArmXLeft, 270) // Lado interno do braço (alinhado ao torso)

   ..quadraticBezierTo(upperArmXLeft - 20, 300, 100, 270) // Curva do antebraço para fora

   ..lineTo(100, 150) // Lado externo do braço para cima

   // Curva suave do ombro de volta para o ponto de conexão

   ..quadraticBezierTo(upperArmXLeft - 25, shoulderY - 10, upperArmXLeft, shoulderY)

   ..close();

  drawPart(canvas: canvas, path: lArmPath, part: lArm);

  

  // --- 4. BRAÇO DIREITO (Conectado ao ombro) ---

  final rArm = parts.firstWhere((p) => p.id == 'r_arm');

  final rArmPath = Path()

   // Começa no ponto do ombro do torso (upperArmXRight, shoulderY)

   ..moveTo(upperArmXRight, shoulderY)

   ..lineTo(upperArmXRight, 270) // Lado interno do braço (alinhado ao torso)

   ..quadraticBezierTo(upperArmXRight + 20, 300, 300, 270) // Curva do antebraço para fora

   ..lineTo(300, 150) // Lado externo do braço para cima

   // Curva suave do ombro de volta para o ponto de conexão

   ..quadraticBezierTo(upperArmXRight + 25, shoulderY - 10, upperArmXRight, shoulderY)

   ..close();

  drawPart(canvas: canvas, path: rArmPath, part: rArm);

  

  // --- 5. PERNA ESQUERDA (Conectada ao quadril) ---

  final lLeg = parts.firstWhere((p) => p.id == 'l_leg');

  final lLegPath = Path()

   // Começa no ponto do quadril esquerdo e crotch do torso (hipInnerXLeft e 200)

   ..moveTo(hipInnerXLeft, crotchY) // Left inner hip

   ..lineTo(200, crotchY) // Center crotch point

   ..lineTo(200, 520) // Inner leg down

   ..quadraticBezierTo(hipInnerXLeft - 20, 550, 140, 520) // Outer foot/ankle curve

   ..lineTo(150, hipY) // Sobe até o lado externo do quadril (para preencher o espaço)

   ..close();

  drawPart(canvas: canvas, path: lLegPath, part: lLeg);

  

  // --- 6. PERNA DIREITA (Conectada ao quadril) ---

  final rLeg = parts.firstWhere((p) => p.id == 'r_leg');

  final rLegPath = Path()

   // Começa no ponto do quadril direito e crotch do torso (hipInnerXRight e 200)

   ..moveTo(hipInnerXRight, crotchY) // Right inner hip

   ..lineTo(200, crotchY) // Center crotch point

   ..lineTo(200, 520) // Inner leg down

   ..quadraticBezierTo(hipInnerXRight + 20, 550, 260, 520) // Outer foot/ankle curve

   ..lineTo(250, hipY) // Sobe até o lado externo do quadril (para preencher o espaço)

   ..close();

  drawPart(canvas: canvas, path: rLegPath, part: rLeg);

  

  // --- 1. CABEÇA (Desenha por último) ---

  final head = parts.firstWhere((p) => p.id == 'head');

  drawPart(canvas: canvas, path: Path(), part: head, isCircle: true);

 }

  

 @override

 bool shouldRepaint(covariant CustomPainter oldDelegate) {

  // Repinta se os dados mudarem (porcentagem ou hover)

  return true;

 }

}

  

// --- Componente Principal (StatefulWidget) ---

  

class BodyMonitorApp extends StatefulWidget {

 const BodyMonitorApp({super.key});

  

 @override

 State<BodyMonitorApp> createState() => _BodyMonitorAppState();

}

  

class _BodyMonitorAppState extends State<BodyMonitorApp> {

 String? hoveredPartId;

  

 // Ajuste das coordenadas de texto para as novas formas

 List<BodyPart> bodyParts = [

  BodyPart(id: 'head', name: 'Cabeça', percentage: 100.0, textX: 200, textY: 75),

  BodyPart(id: 'torso', name: 'Peito/Tronco', percentage: 95.0, textX: 200, textY: 210),

  BodyPart(id: 'l_arm', name: 'Braço Esq.', percentage: 100.0, textX: 125, textY: 210),

  BodyPart(id: 'r_arm', name: 'Braço Dir.', percentage: 55.0, textX: 275, textY: 210), // Amarelo

  BodyPart(id: 'l_leg', name: 'Perna Esq.', percentage: 25.0, textX: 170, textY: 420), // Vermelho

  BodyPart(id: 'r_leg', name: 'Perna Dir.', percentage: 100.0, textX: 230, textY: 420),

 ];

  

 void updateStatus(String id, double value) {

  setState(() {

   final index = bodyParts.indexWhere((p) => p.id == id);

   if (index != -1) {

    // Encontra a parte e atualiza o valor

    bodyParts[index].percentage = value.roundToDouble();

   }

  });

 }

  

 @override

 Widget build(BuildContext context) {

  return MaterialApp(

   debugShowCheckedModeBanner: false,

   theme: ThemeData.dark().copyWith(

    // Simula o bg-slate-900

    scaffoldBackgroundColor: const Color(0xFF0F172A),

    // Cores dos Sliders (Thumb/Active Track)

    sliderTheme: SliderThemeData(

     activeTrackColor: Colors.blue.shade500,

     inactiveTrackColor: Colors.blueGrey.shade600,

     thumbColor: Colors.blue.shade400,

     overlayColor: Colors.blue.shade400.withOpacity(0.2),

    ),

   ),

   home: Scaffold(

    body: SingleChildScrollView(

     child: Center(

      child: Padding(

       padding: const EdgeInsets.all(16.0),

       child: Column(

        crossAxisAlignment: CrossAxisAlignment.center,

        children: [

         // --- Header ---

         const SizedBox(height: 32.0),

         Row(

          mainAxisAlignment: MainAxisAlignment.center,

          children: [

           const Icon(Icons.show_chart, color: Colors.green, size: 30),

           const SizedBox(width: 8),

           const Text(

            'Monitoramento Biométrico',

            style: TextStyle(

             fontSize: 24,

             fontWeight: FontWeight.bold,

             color: Colors.white,

            ),

           ),

          ],

         ),

         const Text(

          'Status atual dos sistemas corporais',

          style: TextStyle(color: Color(0xFF94A3B8), fontSize: 14), // slate-400

         ),

         const SizedBox(height: 32.0),

  

         // --- Conteúdo Principal (Layout Responsivo) ---

         Wrap(

          alignment: WrapAlignment.center,

          spacing: 32.0, // Espaçamento entre os wraps (SVG e Painel)

          runSpacing: 32.0,

          children: [

           // --- Lado Esquerdo: O Desenho CustomPaint ---

           Container(

            width: 350,

            height: 550,

            decoration: BoxDecoration(

             color: const Color(0xFF1E293B).withOpacity(0.5), // slate-800/50

             borderRadius: BorderRadius.circular(24.0),

             border: Border.all(color: const Color(0xFF334155)), // slate-700

             boxShadow: [

              BoxShadow(

               color: Colors.black.withOpacity(0.4),

               blurRadius: 20,

               offset: const Offset(0, 10),

              ),

             ],

            ),

            child: CustomPaint(

             // O CustomPaint usará os dados atualizados

             painter: BodyPainter(bodyParts, hoveredPartId),

             child: const SizedBox(width: 320, height: 500),

            ),

           ),

  

           // --- Lado Direito: Painel de Informações e Controles ---

           SizedBox(

            width: 400,

            child: Column(

             crossAxisAlignment: CrossAxisAlignment.start,

             children: [

              // Controles de Status

              Container(

               padding: const EdgeInsets.all(16.0),

               decoration: BoxDecoration(

                color: const Color(0xFF1E293B), // slate-800

                borderRadius: BorderRadius.circular(16.0),

                border: Border.all(color: const Color(0xFF334155)),

                boxShadow: [

                 BoxShadow(

                  color: Colors.black.withOpacity(0.2),

                  blurRadius: 10,

                 ),

                ],

               ),

               child: Column(

                crossAxisAlignment: CrossAxisAlignment.start,

                children: [

                 Row(

                  children: [

                   Icon(Icons.security, color: Colors.blue.shade400, size: 20),

                   const SizedBox(width: 8),

                   const Text(

                    'Controle de Status',

                    style: TextStyle(

                      fontSize: 18,

                      fontWeight: FontWeight.w600,

                      color: Colors.white),

                   ),

                  ],

                 ),

                 const Divider(color: Color(0xFF334155), height: 20),

                 const Text(

                  'Ajuste os sliders para testar as cores (Verde ≥ 60%, Amarelo < 60%, Vermelho < 30%).',

                  style: TextStyle(

                    fontSize: 12, color: Color(0xFF94A3B8)),

                 ),

                 const SizedBox(height: 16),

                 // Lista de Sliders

                 ...bodyParts.map((part) =>

                   MouseRegion(

                    onEnter: (_) => setState(() => hoveredPartId = part.id),

                    onExit: (_) => setState(() => hoveredPartId = null),

                    child: Container(

                     padding: const EdgeInsets.symmetric(horizontal: 12.0, vertical: 8.0),

                     margin: const EdgeInsets.only(bottom: 8.0),

                     decoration: BoxDecoration(

                      color: hoveredPartId == part.id

                        ? const Color(0xFF334155) // slate-700

                        : const Color(0xFF1E293B).withOpacity(0.5),

                      borderRadius: BorderRadius.circular(8.0),

                      border: Border.all(

                       color: hoveredPartId == part.id

                         ? Colors.blue.shade400

                         : const Color(0xFF334155),

                      ),

                     ),

                     child: Column(

                      crossAxisAlignment: CrossAxisAlignment.start,

                      children: [

                       Row(

                        mainAxisAlignment: MainAxisAlignment.spaceBetween,

                        children: [

                         Text(

                          part.name,

                          style: const TextStyle(

                            fontSize: 14, color: Color(0xFFE2E8F0), fontWeight: FontWeight.w500),

                         ),

                         Text(

                          '${part.percentage.toInt()}%',

                          style: TextStyle(

                           fontSize: 16,

                           fontWeight: FontWeight.bold,

                           color: part.percentage < 30

                             ? Colors.red.shade400

                             : part.percentage < 60

                               ? Colors.yellow.shade400

                               : Colors.green.shade400,

                          ),

                         ),

                        ],

                       ),

                       Slider(

                        value: part.percentage,

                        min: 0,

                        max: 100,

                        divisions: 100,

                        onChanged: (double newValue) {

                         updateStatus(part.id, newValue);

                        },

                       ),

                      ],

                     ),

                    ),

                   ),

                 ).toList(),

                ],

               ),

              ),

  

              const SizedBox(height: 16),

  

              // Resumo da Legenda

              Container(

               padding: const EdgeInsets.all(16.0),

               decoration: BoxDecoration(

                color: const Color(0xFF1E293B).withOpacity(0.8),

                borderRadius: BorderRadius.circular(16.0),

                border: Border.all(color: const Color(0xFF334155)),

               ),

               child: Column(

                crossAxisAlignment: CrossAxisAlignment.start,

                children: [

                 Row(

                  children: [

                   Icon(Icons.favorite, color: Colors.pink.shade500, size: 18),

                   const SizedBox(width: 8),

                   const Text(

                    'Resumo',

                    style: TextStyle(

                      fontSize: 16,

                      fontWeight: FontWeight.w600,

                      color: Colors.white),

                   ),

                  ],

                 ),

                 const SizedBox(height: 8),

                 Wrap(

                  spacing: 8.0,

                  runSpacing: 8.0,

                  children: [

                   _buildLegendItem('Saudável (≥60%)', Colors.green),

                   _buildLegendItem('Atenção (<60%)', Colors.yellow),

                   _buildLegendItem('Crítico (<30%)', Colors.red),

                  ],

                 ),

                ],

               ),

              ),

             ],

            ),

           ),

          ],

         ),

         const SizedBox(height: 32.0),

        ],

       ),

      ),

     ),

    ),

  )

  );

 }

  

 // Widget auxiliar para a legenda

 Widget _buildLegendItem(String text, MaterialColor color) {

  return Container(

   padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 5),

   decoration: BoxDecoration(

    color: color.withOpacity(0.2),

    borderRadius: BorderRadius.circular(6.0),

    border: Border.all(color: color.withOpacity(0.3)),

   ),

   child: Text(

    text,

    style: TextStyle(color: color.shade400, fontSize: 12),

   ),

  );

 }

}

  

void main() {

 runApp(const BodyMonitorApp());

}