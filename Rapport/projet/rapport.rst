.. footer:: page ###Page###

================================
Rapport de Projet **Train** FPGA
================================

-----------------------------
FPGA 1 Syst�mes programmables
-----------------------------

|
|
|
|

*Ayrault Maxime* **3203694** - *Nguyen Gabriel* **3200625**

|
|
|
|
|
|
|

----------------------------------------------------------

Introduction
============

|
|

|

| Ce document pr�sente le rapport du projet train de l'UE *FPGA1*.
| Le projet a pour but de r�aliser le circuit electrique d'une *centrale DCC* et de la 
| tester sur une vraie maquette. 
| Lors de ce Projet nous nous sommes servis du logiciel **Vivado** pour le developpement de la
| centrale ainsi que la carte **FPGA** ``Nexys 4DDR``.
| Lors de ce projet nous avons aussi decouvert le protocole DCC, que nous avons immpl�ment�.
|
|

Ce projet ce decoupe en 2 �tapes qui correspondent � 2 types de centrales :
 - une uniquement en ***vhdl***.
 - l'autre en ***vhdl*** ainsi qu'en ***C*** en utilisant le microblaze de la carte. 

bla bla...

Ce projet � �t� devellop� sous git :
 https://github.com/maximouth/Projet_FPGA

|
|
|
|
|
|
|
|

 

--------------------------------------------



I) Protocole DCC
================


|
| Le protocole DCC est un protocole standardis� qui permet de communiquer entre la carte 
| **FPGA** et les diff�rents trains et �quipements de voies.
| Il utilise une suite de commandes envoy�es sur les rails jusqu'aux diff�rents trains et 
| composants qui agissent en fonction de la commande qu'ils reçoivent. La locomotive peut 
| recevoir �norm�ment de commandes diff�rentes, klaxon, annonces d'entr�e de gare, phares... 
| (voir datasheet locomotive). Elles ne seront pas toutes implement�es ici, mais pourront être
| rajout�es ult�rieurement. 
|
|


.. image:: trame.png
   :scale: 250 %
   :alt: trame protocale DCC
   :align: center

|


Cette image represente une trame DCC et son contenu.
Chaque trame est compos�e de cette fa�on :

 - 14 *bit* � '1' *(preamble)*
 - 1 *octet* **d'adresse** 
 - 1 *octet* de **data**
 - 1 *octet de **CRC** *Xor* entre *adresse* et *data* *((epilogue))*

Chaque partie est s�par�e par un *bit* � '0'.

|
|
|
|
|
|
|
|
|
|
|
|
|
|
|
|
|
|
|

--------------------------------------------------------------------

|
|

II) Architecture
================

|

1) VHDL
#######

|

.. image:: schema_vhdl.png
   :scale: 50 %
   :alt: trame protocale DCC
   :align: center

|
|

| Nous avons commenc� par cr�er une centrale DCC uniquement en version mat�rielle avec
| uniquement du VHDL. L'architecture r�alis�e est plutot simple et est compos�e de diff�rents
| �l�ments que nous allons vous detailler plus loin.


|
|

2) VHDL & C
###########

|

.. image:: schema_vhdl_c.png
   :scale: 50 %
   :alt: trame protocale DCC
   :align: center

|
|

| Nous avons ensuite modifi� la premi�re centrale mat�rielle pour utiliser le microblaze de la carte.
| Le microblaze sert � gerer les diff�rents appuis sur les boutons de l'IHM.

|
|
|
|
|
|

---------------------------------------


III) Fonctions
==============

|

1) Clock Divider
################


|

.. code:: VHDL

 entity div_clock is
  Port (
    -- 100 MHz signla
    clk : in STD_LOGIC;
    -- 1 MHz signal
    div_clock : out STD_LOGIC);
 end div_clock;
	  
	  

| La carte nexys 4 DDR tourne � 100Mhz ce qui n'est pas pratique pour gerer des signaux en
| sortie qui doivent �tre en us.
| Pour simplifier le gestion du temps nous avons creer un diviseur d'horloge. Qui diminue
| la vitesse de 100 MHz � 1 MHz.
| Ce qui facilite l'utilisation du protocole DCC.
|

.. image:: trame/div_clock.png
   :scale: 250 %
   :alt: trame protocale DCC
   :align: center

|
|
| On voit que sur la simulation, la periode de la sortie du module est 100x plus petite que
| celle de l'entr�e.
| On divise bien l'horloge par 100, pour passer de 100 MHz � 1 MHz.
|

2) Send_One
################

|

.. code:: VHDL

 entity send_one is
  Port (
    clk     : in  STD_LOGIC;
    start_1 : in  STD_LOGIC := '0';
    end_1   : out STD_LOGIC := '0';
    pulse_1 : out STD_LOGIC := '0'
    );      
 end send_one;




| Ce petit module sert � envoyer un *'1'* en suivant le protocole **DCC**, c'est � dire envoyer un *'0'* 
| logique pendant **58** clock cycles suivit d'un *'1'* logique pendant **58** clock cycles.
| Il tourne � *1 Mhz* gr�ce au module ``clock_divider``.
|

.. image:: trame/send_one.png
   :scale: 250 %
   :alt: trame protocale DCC
   :align: center

|
|
| On observe sur la simulation que � partir du moment ou le signal ``start_1`` passe � *'1'* le signal de 
| sortie envoie un *'0'* pendant **58**  cycles suivit d'un *'1'* pendant **58** cycles et signal que l'envoie est
| termin� par le signal ``end_1`` �  *'1'* pendant **1** cycle.

3) Send_Zero
################

|

.. code:: VHDL

 entity send_zero is
  Port (
    clk     : in  STD_LOGIC;
    start_0 : in  STD_LOGIC := '0';
    end_0   : out STD_LOGIC := '0';
    pulse_0 : out STD_LOGIC := '0'
    );      
 end send_zero;
	  

| Ce petit module sert � envoyer un *'0'* en suivant le protocole **DCC**, c'est � dire envoyer un *'0'* 
| logique pendant **100** clock cycles suivit d'un *'1'* logique pendant **100** clock cycles.
| Il tourne � *1 Mhz* gr�ce au module ``clock_divider``.
|

.. image:: trame/send_zero.png
   :scale: 250 %
   :alt: trame protocale DCC
   :align: center

|
|
| On observe sur la simulation que � partir du moment ou le signal ``start_0`` passe � *'1'* le signal de 
| sortie envoie un *'0'* pendant **100**  cycles suivit d'un *'1'* pendant **100** cycles et signal que l'envoie est
| termin� par le signal ``end_0`` �  *'1'* pendant **1** cycle.
|

4) Send_preamble
################


|

.. code:: VHDL

 entity send_preamble is
  Port (
    clk     : in  STD_LOGIC;
    start_p : in  STD_LOGIC := '0';
    end_p   : out STD_LOGIC := '0';
    pulse_p : out STD_LOGIC := '0'
    );      
 end send_preamble;
	  
	  

| Ce module sert � envoyer un preambule en suivant le protocole **DCC**, c'est � dire envoyer 
| une suite de 14 *'1'*. Ce module se sert du petit module ``send_one``.
| Ce module attend de recevoir un signal **start_p** qui lui signal qu'il doit envoyer un preambule. 
| Il se sert d'un compteur initialis� � ``0`` qui sert � connaitre le nombre de *'1'* envoy�. 
| Il envoie  un **start_1** au module ``send_one`` et  attend de recevoir le signal  **end_1**
| pour incrementer le compteur.
| Une fois le preambule envoy� il renvoie le signal **end_p** qui signifie qu'il a fini.
|

.. image:: trame/send_preamble.png
   :scale: 250 %
   :alt: trame protocale DCC
   :align: center

|
|
| On observe sur la simulation qu'une fois le signal ``start_p`` passe � *'1'* le signal de sortie 
| envoie **14** *'1'* suivant le protocole DCC.
| Une fois les  **14** *'1'* envoy�, il signale qu'il a fini avec le signal ``end_p`` � *'1'* pendant **1** cycle.
|
|

5) Send_byte
################

|

.. code:: VHDL

 entity send_byte is
  Port (
    clk     : in  STD_LOGIC;
    start_b : in  STD_LOGIC := '0';
    byte    : in  Std_Logic_Vector (7 downto 0); 
    end_b   : out STD_LOGIC := '0';
    pulse_b : out STD_LOGIC := '0'
    );      
 end send_byte;

	  
|
|
| Ce module sert � envoyer un octet en suivant le protocole **DCC**, c'est � dire envoyer 
| une suite de 8 *'1'* ou *'0'* selon la valeur de l'octet en entr�e. Ce module se sert des
| petits modules ``send_one``, et ``send_zero``.
| Ce module attend de recevoir un signal **start_b** qui lui signal qu'il doit envoyer un octet. 
| Il se sert d'un compteur initialis� � ``0`` qui sert � connaitre le nombre de *bit(s)* envoy�s. 
| Il envoie  un **start_0/1** � l'un des deux sous module et  attend de recevoir le signal
| **end_0/1** avant d'envoyer le bit suivant et incrementer le compteur.
| Une fois l'octet envoy� il renvoie le signal **end_b** qui signifie qu'il a fini.
|

.. image:: trame/send_byte.png
   :scale: 250 %
   :alt: trame protocale DCC
   :align: center

|
|
| Lors de cette simulation nous cherchons � envoyer l'octet suivant *"10101010"*, c'est ce que nous
| avons mit dans le signal ``byte``.
| On observe sur la simulation qu'une fois le signal ``start_b`` passe � *'1'* le signal de sortie 
| envoie les diff�rentes valeurs contenue dans le signal ``byte`` c'est � dire une alternance de *'1'* et de *'0'*.
| Une fois les diff�rents bits de l'octet envoy�, il signale qu'il a fini avec le signal ``end_b`` � *'1'* pendant **1** cycle.
|
|


6) Sequencer
################



|

.. code:: VHDL

 entity sequencer is
  Port (
    clk       : in  STD_LOGIC;

    go        : in  STD_LOGIC := '0';
    
    addr      : in  Std_Logic_Vector (7 downto 0);
    feat      : in  Std_Logic_Vector (7 downto 0);
    speed     : in  Std_Logic_Vector (7 downto 0);
    which     : in  Std_Logic;
    idle      : in  Std_Logic;    

    done      : out Std_Logic;
    pulse     : out STD_LOGIC := '0'
    );      
 end sequencer;

| Ce module est implement� gr�ce � une machine à �tats, qui va g�rer l'envoie des 4 trames
| *(Idle, Vitesse, Fonction, Idle)*.


|
|
|
|
|
|
|
|
|

--------------------------------------------------

|
|

IV) IHM
=======

|
|


.. image:: exe_add.jpg
   :scale: 150 %
   :alt: photo de l'interface
   :align: center


|
|                                      *Photo de l'interface*
|
|
| Voici une photo de l'*IHM* de notre projet.

Les *afficheurs 7 segments* sont decoup�s en 2 :

 - Les 4 de **gauches** servent � afficher le nom de la commande. 
 - Les 4 de **droites** servent � afficher la valeur de la commande. 

|

On utilise **3** *boutons* sur les 5 :

 - Le bouton de **gauche** pour changer la commande.
 - Le bouton de **droite** pour incrementer la valeur de la commande
   affich�e.
 - Le bouton du **milieu** pour envoyer les nouvelles valeurs entr�es vers
   le module ``Master``

|

Le switch de droite sert de reset, c'est un reset actif haut.   


|
|
|


.. code:: VHDL

 entity control_seg is
  Port ( CLK    : in STD_LOGIC;
         reset  : in STD_LOGIC;
         CA     : out STD_LOGIC;
         CB     : out STD_LOGIC;
         CC     : out STD_LOGIC;
         CD     : out STD_LOGIC;
         CE     : out STD_LOGIC;
         CF     : out STD_LOGIC;
         CG     : out STD_LOGIC;
         DP     : out STD_LOGIC;
         AN     : out STD_LOGIC_VECTOR (7 downto 0);
         ADD    : out STD_LOGIC_VECTOR (7 downto 0);
         SPD    : out STD_LOGIC_VECTOR (7 downto 0);
         FEAT   : out STD_LOGIC_VECTOR (7 downto 0);
         -- chose setting
         BTNL   : in STD_LOGIC;
         -- increment setting value
         BTNR   : in STD_LOGIC
         );
 end control_seg;

|
|
| Voici l'interface de notre IHM.
| Elle re�oit en entr�e la clock et le reset, ainsi que la valeur des diff�rents boutons.
| Et en sortie les diff�rents fils servant � contr�ler les afficheurs *CA*->*CG* *DP* *AN* ainsi que
| les valeurs courantes des octets d'adresse, de vitesse et de la commande.
|
|

-------------------------------

|
|

V) Impl�mentation sur la maquette
=================================

explication comment interfacer interface et la centrale.
(schema, et explication?

.. code:: VHDL

 entity Master is
  Port ( CLK   : in STD_LOGIC;
         BTNC  : in STD_LOGIC;
         BTNL  : in STD_LOGIC;
         BTNR  : in STD_LOGIC;
         reset : in STD_LOGIC;
         LED   : out STD_LOGIC;
         CA    : out STD_LOGIC;
         CB    : out STD_LOGIC;
         CC    : out STD_LOGIC;
         CD    : out STD_LOGIC;
         CE    : out STD_LOGIC;
         CF    : out STD_LOGIC;
         CG    : out STD_LOGIC;
         DP    : out STD_LOGIC;
         AN    : out Std_Logic_Vector (7 downto 0);
         PULSE : out STD_LOGIC);

 end Master;

	  

| 
| Le module ``interface`` envoie en permanence vers le module ``Master`` la valeur que lui a des diff�rentes commandes.
| Le module ``Master`` ne mets � jour les valeurs � envoyer vers le train que lorsqu'il detecte un appuit
| sur le bouton central. Il met � jour la valeur locale des commandes.
|
Il envoit par contre en continue un groupe de 4 trames vers le(s)
train(s) :

 - IDLE  : Ne fait rien
 - SPEED : Envois la valeur de la vitesse au train choisit
 - FEAT  : Envois la commande au train choisit
 - IDLE  : Ne fait rien

|
| Ce module sert en fait � disperser les differents fils en entr�e ou sortie vers les differents composants.
|
|

image oscilloscope

.. image:: envois_feat.png
   :scale: 100 %
   :alt: envoie vitesse
   :align: center

|
| envois trame adresse 0 et vitesse
|
	   
.. image:: envois_idle.png
   :scale: 100 %
   :alt: envoit data
   :align: center

|
| envois trame idle
|

      
fonctionalit�es impl�ment�es (type de vitesse, phares, klakons)
tableau


image maquette

.. image:: maquette.png
   :scale: 100 %
   :alt: photo maquette
   :align: center


VI) Microblaze
==============



VII) Conclusion
===============
