[Flujo de trabajo](https://www.figma.com/board/qG8L53L90LqxQwhcNhphPx/AKO-16523PN?node-id=0-1&p=f&t=OGXa0Z6zLPEFhqYc-0)

Sacar Tokens_

Publicos y privados 

![[Pasted image 20260318153610.png]]

`public: NCJu1214hzjVOv4j4JtlMLqIk7ts1xvrgMtkEjthtkEENcK3lx`
`private: R6dE1qWVYgsX4FPXkW76kJT9y5GtRqXQjNMK1oUhhXpkfuNKvD`

El de _company_ se saca de una cuenta con privilegios (_Fatin_) Cojo el ObjectId de la empresa y en la coleción de ´apikeys´ 

`db.getCollection("apikeys").find({_company:ObjectId("5ab8a13370f53c000bc46384")})`

`company: `




