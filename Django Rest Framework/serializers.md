Django Rest Framework (DRF) Serializers
-------------------

Los serializers son clases usadas para validar y preparar datos de entrada de los servicios REST, manteniendo el código separado y reutilizable. Desués de mandar los datos de entrada a un serializador, este nos debe indicar si hubo algún error en algún campo, cual campo y un mensaje para el usuario. Si no hubo ningún error, entonces los datos deben ya haber recibido el ajuste o transformación necesarios para poder usar los valores internamente en el servicio. El serializador también se usa para des-serializar los datos.

Un serializador se compone de campos que son los que contienen los datos a procesar. Cada campo se maneja también como una clase que es la que se encarga de procesar la información relacionada a dicho campo. DRF ya incorpora varios campos de uso común, algunos de estos son los siguientes:
* ``









