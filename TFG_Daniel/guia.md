# Pequena guía do TFG

## Obxectivos

Simulación da reacción de transferencia $^{11}{Li}(d,t)^{10}{Li}$, implementando:

- Xeometría dos detectores (require adaptación)
- Perdas de enerxía e straggling (xa feito)
- Cinemática exacta (parcialmente feito)
- Sección eficaz teórica
- Outras variables que nos poidan ir parecendo interesantes

Para obter o seguinte:

- Espectro en enerxía de excitación para os primeiros estados excitados do $^{10}{Li}$
- Ver como queda o espectro final

## Configuración experimental

Lembra que debemos configurar o detector da seguinte maneira:

- O gas de ACTAR TPC será unha mestura de 95% D$_2$ e 5% iC$_4$H$_{10}$
a unha presión de 900 mbar. As táboas de SRIM deben reflectir isto.
- O feixe incidente de $^{11}Li$ terá unha enerxía de 7,5 AMeV.
- Vamos traballar con dous muros de silicios situados na parte dianteira,
de forma que cubran un gran ángulo sólido. Medirán a partícula lixeira (t) da reacción.

## Para comezar

1. Xa sabes resolver a cinemática para a 4a partícula. Fai o mesmo para a terceira,
pois é a que habitualmente se mide.
2. Permite unha **excitación** na partícula pesada (4): $m_4 = m_4 + E_x$.
  O teu macro debe aceptar diferentes $E_x$, xa que as iremos variando.
3. Verifica os resultados da túa clase de cinemática coa miña `ActPhysics::Kinematics`:
representa a enerxía $T_{3,4}$ vs $\theta_{3,4; Lab}$ e contrasta cos meus resultados.

## Simulación básica

Utiliza [o macro guía](https://github.com/loopset/e796-analysis/blob/main/Simulation/TFG/simulation.cxx) que che preparei
para ter unha base. É escolla túa se queres usar
a túa clase de cinemática ou `ActPhysics::Kinematics`, sempre e cando te asegures que
funciona ben.

O algoritmo base é:

1. Samplea un vértice. Debe estar centrado en YZ e X $\in [0, 256]$ mm.
2. Corrixe a enerxía inicial do beam ata ese punto con SRIM.
3. Samplea **uniformemente** (de momento) $\theta_{CM}$ e $\phi_{CM}$.
Xera a cinemátic no Lab.
4. Define o vector dirección da partícula lixeira.
5. Comproba se é capaz de impactar nun silicio da primeira layer.
6. Se si, propaga a partícula:
    - Primeiro, aplica unha resolución no ángulo da partícula lixeira.
    - Despois, reduce a enerxía da partícula dende o vértice ata o silicio.
    Usa `ActPhysics::SRIM::SlowWithStraggling`, que automaticamente implementa o straggling.
    - Fai o mesmo no silicio. Teñen un espesor de 1500 $\mu$m. Aplica a resolución.
7. Agora temos que facer á inversa: experimentalmente partimos da enerxía no Si
para reconstruír a enerxía no vértice.
    - Sabes a distancia que atravesou ata o silicio. Recupera a enerxía no vértice
    con `ActPhysics::SRIM::EvalInitialEnergy`.
    - Presta atención ao **punch-through**: cando a partícula ten demasiada enerxía
    e non se para no silicio.
    - Por último, reconstrúe a **enerxía de excitación**. Se todo foi ben, deberías
    ver que non é unha $\delta$ centrada no valor que especificaches, senón
    que é unha gaussiana cunha $\sigma$ determinada.

8. Finalmente, fai representacións gráficas:
    - A cinemática sampleada (`TH2D`).
    - A reconstruída (`TH2D`).
    - O $\theta_{CM}$ sampleado (`TH1D`).
    - O punto de impacto no Si (`TH2D`).
    - O espectro de $E_x$ (`TH1D`).

## Marco teórico
Estudar o Li10 danos pistas sobre o estado fundamental do Li11, un dos núcleos lixeiros máis coñecidos posto que presenta unha estrutura de **halo**.

O $^{10}$Li é un núcleo **non ligado** ($S_n < 0$), polo que se vai formar como resonancia na reacción,
tendo unha anchura característica $\Gamma$ (á parte da $\sigma$ experimental habitual).
Determinar o carácter de cada un dos estados resonantes do $^{10}$Li é esencial para saber cal
é a función de ondas do $^{11}$Li:
$$^{11}{\text{Li}} = ^{10}{\text{Li}}\; \otimes\; \nu(1p_{1/2}\;  \text{ou}\; 2s_{1/2}) $$

O que quere dicir que, determinando o momento angular $\ell = 0,1$ da resonancia, podemos inferir que configuración
tomaba o neutrón arrancado do $^{11}$Li.

Aquí algunha bibliografía interesante:

- [Estudo previo](https://doi.org/10.1016/j.physletb.2016.02.060) usando $(p,d)$, equivalente ao noso $(d,t)$.
- [Cálculos teóricos](https://doi.org/10.1016/j.physletb.2017.02.017) para ese experimento (Fig. 3).
