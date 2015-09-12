---
layout: post
title:  "React Webpack Starter"
date:   2015-08-26 21:35:01
description: Mise en place d'une stack de développement efficace avec React / Webpack, en profitant de la syntaxe ES6 et d'un outil de qualité de code (ESLint).
categories:
- blog
permalink: react-webpack-starter
---
Mettre en place une stack de développement efficace et rapide pour travailler avec React ça prend un peu de temps. Bénéficier du hot reload, charger correctement ses ressources statiques, linter son code ou encore utiliser la nouvelle syntaxe ES6 c'est autant d'éléments à configurer.

Avec Webpack c'est pas sorcier mais c'est souvent synonyme d'heures perdues à mettre en place l'ensemble.

Deux façons donc de lire cet article :

- vous êtes déjà familier de cet environnement et vous pouvez directement cloner le projet **react-webpack-starter** dispo sur mon [github][github] pour commencer votre nouvelle application.
- vous démarrez sur React et/ou Webpack, vous pouvez suivre pas à pas cet article pour comprendre la stack mise en place.

Enjoy :)

## Setup React
Pour gérer nos dépendances, on utilise `npm`. On démarre le projet :
{% highlight console %}
$ npm init
{% endhighlight %}

Il suffit de répondre oui (yes) à toute les questions pour initier le **package.json** à la racine du projet. <br>

On installe ensuite notre première dépendance, React :
{% highlight console %}
$ npm install react --save
{% endhighlight %}

Avant d'aller plus loin, découvrons l'arborescence de notre projet React :

    - app
        - App.jsx
        - index.js
    - build
        - bundle.js **contenu généré**
        - index.html

Passons rapidement en revue ces 3 fichiers :

**index.html**
{% highlight html %}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>React Webpack Starter</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="stylesheet" type="text/css" href="styles.css">
    </head>
    <body>
        <div id="root"></div>
        <script src="bundle.js"></script>
    </body>
</html>
{% endhighlight %}

`index.html` c'est le fichier de départ chargé par le navigateur web. Lorsque l'on va lancer Webpack, ce dernier va compiler tous nos scripts JS dans le seul fichier `bundle.js`.

**index.js**
{% highlight javascript %}
import React from 'react';
import App from './App';

React.render(
    <App />,
    document.getElementById('root')
);
{% endhighlight %}

Le point d'entrée de l'application React c'est le fichier `index.js`. Il inclut notamment le composant parent `App`. Ce dernier est notre composant React de plus haut niveau, qui contiendra les autres sous-composants. Le contenu du fichier `index.js` permet de faire le pont avec le fichier `index.html`. En effet, c'est dans la méthode *render* qu'on s'attache au DOM sur la div portant l'id **root** (qui se trouve dans notre `index.html`).

**App.jsx**
{% highlight javascript %}
import React from 'react';

export default class App extends React.Component {

    render() {
        return (
            <h1>Hello world</h1>
        );
    }

}
{% endhighlight %}

Enfin, `App`, notre composant React parent qui inclura tous les autres. Oui oui, c'est bien du JavaScript mais à la sauce ES6. Notre composant est donc une classe qui étend la classe `React.Component`. Sa seule méthode, **render**, ne retourne pour l'instant qu'un simple *Hello World*. Enfin, pour simplifier l'écriture on utilise la syntaxe `JSX` pour écrire nos composants React.

## Wepack
Ok, le minimum vital pour React est présent, on enchaine maintenant avec [Webpack][webpack] :

{% highlight console %}
$ npm install webpack --save-dev
{% endhighlight %}

Difficile de décrire Webpack en 2 lignes. C'est avant tout un outil pour charger vos scripts JavaScript en tant que module, peu importe leur syntaxe (CommonJS, AMD). Grâce à différents loaders qu'on peu lui ajouter, Webpack est capable de consommer tout ce qui ressemble de près ou de loin à un fichier (CSS, images, json, font, etc). C'est à la fois un super *task runner* et un gestionnaire de modules universel. Imbattable.

Wepack se paramètre grâce à un fichier de config, `webpack.config.js`, qui se place à la racine de votre projet.

**webpack.config.js**
{% highlight javascript %}
var path = require("path");
var webpack = require("webpack");

module.exports = {
    entry: ['./app/index.js'],
    output: {
        path: path.join(__dirname, './build'),
        filename: 'bundle.js',
        publicPath: '/'
    },
    module: {
        loaders: []
    },
    resolve: {
        extensions: ['', '.js', '.jsx']
    }
};
{% endhighlight %}

Passons en revue les différentes lignes :

- entry : le point d'entrée de votre application
- output.path : le répertoire de sortie utilisé par Webpack
- output.filename : le fichier généré qui contient tous nos scipts JS concaténés
- publicPath : la *base url*
- module.loaders : un tableau contenant les différents loaders
- resolve.extensions : les extensions à résoudre par Webpack. On n'oublie pas de préciser l'extension `JSX` qu'on utlilise pour React.

On poursuit avec Webpack en lui ajoutant différents loaders.

#### Babel
Aujourd'hui la syntaxe ES6 n'est pas encore comprise par tous les navigateurs internet, on va donc utiliser [babeljs][babeljs] pour transpiler notre code en ES5. Cela va permettre d'utiliser les [nouvelles features ES6][features] dès aujourd'hui !

{% highlight console %}
$ npm install babel-loader --save-dev
$ npm install babel-runtime --save
{% endhighlight %}

On complète notre `webpack.config.js` avec ce nouveau loader fraichement installé :
{% highlight javascript %}
...
loaders: [
    {
        test: /\.jsx?$/, // cible tous les fichiers en .jsx
        exclude: /node_modules/, // on exclut le répertoire node_modules
        loader: 'babel?optional[]=runtime' // et on précise quelques options
    }
]
...
{% endhighlight %}

#### CSS/SASS
Le chargement des CSS nécessite deux loaders : `css-loader` et `style-loader`. Le premier a pour but de résoudre les `url(...)` tandis que le second injecte directement les CSS dans une balise style au sein du body de la page. On ne va pas se priver d'écrire en SASS, on complète donc avec le `sass-loader` capable de gérer ce format et d'assurer la compilation. Par défaut, Webpack charge en inline toutes les feuilles de style. Cela donne du code très difficile à débugger. Heureusement, grâce au plugin `ExtractTextPlugin` il est à la fois possible d'extraire nos CSS dans un seul fichier externe et d'activer les *sources maps* pour s'y retrouver.

On installe les 3 loaders :
{% highlight console %}
$ npm install sass-loader css-loader style-loader --save-dev
$ npm install extract-text-webpack-plugin --save-dev
{% endhighlight %}

Et on met à jour notre config Webpack :
{% highlight javascript %}
...
devtool: 'source-map',
module: {
    loaders: [
        {
            test: /\.scss$/,
            loader: ExtractTextPlugin.extract(
                // activate source maps via loader query
                'css?sourceMap!' +
                'sass?sourceMap'
            )
        }
    ]
},
plugins: [
    // extract inline css into separate 'styles.css'
    new ExtractTextPlugin('styles.css')
]
...
{% endhighlight %}

#### Images et fonts
Pour le chargement des images et des fonts on va utiliser `url-loader`. On lui passe en paramètre une limite de poids en dessous de laquelle le loader charge le fichier en base 64 pour un gain de performance.
{% highlight console %}
$ npm install url-loader --save-dev
{% endhighlight %}

**webpack.config.js**
{% highlight javascript %}
...
loaders: [
    {
        test: /\.(png|woff|woff2|eot|ttf|svg)$/,
        loader: 'url-loader?limit=100000'
    }
]
...
{% endhighlight %}

## Unleash the power !
On va enfin pouvoir s'atteler à faire tourner notre projet en local. Pour ce faire, Webpack fournit [webpack-dev-server][webpack-dev]. C'est un serveur Express (NodeJS) qui va nous permettre de servir tous nos fichiers
statiques. On est sur un mode complétement différent de Gulp ou Grunt qui eux utilisent le système de fichiers via des tâches de copier/coller.

{% highlight console %}
$ npm install webpack --save-dev
{% endhighlight %}

Après l'avoir installé, on choisit un numéro de port inutilisé, ici `8080`, et on ajoute un script au **package.json** :
{% highlight javascript %}
...
"scripts": {
  "start": "webpack-dev-server --port 8080 --content-base build/"
}
...
{% endhighlight %}

On lance enfin l'ensemble :
{% highlight console %}
$ npm start
{% endhighlight %}

Et on accède à [http://localhost:8080/][localhost]. <br>
TADAAaaaa ! Un joli *Hello World* :)

#### Hot-reload
La feature ultime de Webpack c'est le Hot-reload. Il est en effet capable de détecter les changements dans votre code et de relancer l'ensemble de l'application. C'est un puissant live-reload capable même de conserver l'état de vos composants React !

Pour l'activer, il faut d'abord ajouter au fichier **index.html** la ligne suivante, juste avant le chargement du script `bundle.js` :
{% highlight html %}
...
<script src="/webpack-dev-server.js"></script>
...
{% endhighlight %}

Et on complète avec deux nouvelles options notre script `npm start` dans le **package.json** :
{% highlight javascript %}
...
"start": "webpack-dev-server --port 8080 --hot --inline --content-base build/"
...
{% endhighlight %}

#### Eslint
Pour éviter les erreurs et les bugs, il est conseillé d'utliser un outil de qualité de code. Le nouveau venu c'est [ESLint][eslint] et il a de sérieux atouts. Compatible avec la syntaxe ES6 et le format JSX de React, il est aussi entièrement configurable.

Pour en profiter pleinement, voici les éléments à installer :

- eslint : l'outil lui-même
- babel-eslint : un parser custom pour s'assurer qu'ESLint soit capable de lire correctement le code transpilé par Babel
- eslint-plugin-react : un plugin pour ajouter des règles spécifiques à la syntaxe JSX de React.
- eslint-loader : le loader adéquat pour exécuter ESLint au lancement de Webpack.

{% highlight console %}
$ npm install eslint babel-eslint eslint-plugin-react eslint-loader --save-dev
{% endhighlight %}

ESLint se base sur un fichier de configuration pour définir les différentes règles. Ce fichier c'est le `.eslintrc` et on le place en général à la racine de son projet (comme le `webpack.config.js`).

**.eslintrc**
{% highlight javascript %}
{
    "env": {
        "browser": true,
        "node": true,
        "es6": true
    },
    "plugins": [
        "react"
    ],
    "parser": "babel-eslint",
    "ecmaFeatures": {
        "jsx": true,
        "modules": true
    },
    "rules": {
        //ESLint rules
        "quotes": [2, "single"],
        "no-unused-vars": 2,

        //React rules
        "react/display-name": 0, // Prevent missing displayName in a React component definition
        "react/jsx-quotes": [2, "double", "avoid-escape"], // Enforce quote style for JSX attributes
        "react/jsx-no-undef": 2, // Disallow undeclared variables in JSX
        "react/jsx-sort-props": 0, // Enforce props alphabetical sorting
        "react/jsx-uses-react": 2, // Prevent React to be incorrectly marked as unused
        "react/jsx-uses-vars": 2, // Prevent variables used in JSX to be incorrectly marked as unused
        "react/no-did-mount-set-state": 2, // Prevent usage of setState in componentDidMount
        "react/no-did-update-set-state": 2, // Prevent usage of setState in componentDidUpdate
        "react/no-multi-comp": 0, // Prevent multiple component definition per file
        "react/no-unknown-property": 2, // Prevent usage of unknown DOM property
        "react/prop-types": 2, // Prevent missing props validation in a React component definition
        "react/react-in-jsx-scope": 2, // Prevent missing React when using JSX
        "react/self-closing-comp": 2, // Prevent extra closing tags for components without children
        "react/wrap-multilines": 2, // Prevent missing parentheses around multilines JSX
    }
}
{% endhighlight %}

Par défaut ESLint intègre quelques règles qu'il est possible d'overrider dans le fichier `.eslintrc`. Chaque règle est suivie d'un numéro : 0 = règle désactivée, 1 = warning, 2 = erreur bloquante. N'hésitez pas à consulter la [liste des règles][rules-eslint].

Il faut maintenant exécuter ces règles côté Webpack. À la place d'un loader, on utilise un preloader, on s'assure ainsi que ESLint s'exécute avant toute autre opération (transpilation, compilation, etc etc).

**webpack.config.js**
{% highlight javascript %}
...
module: {
    preLoaders: [
        {
            test: /\.jsx?$/,
            loader: 'eslint-loader',
            exclude: /node_modules/
        }
    ]
}
...
{% endhighlight %}

Maintenant, il suffit d'ajouter du code qui ne respecte pas ces règles pour voir la magie opérer :) <br>
Par exemple, une variable inutilisée ou des simple quotes dans les attributs JSX donnent dans la console :
{% highlight console %}
...
ERROR in ./app/App.jsx

/Users/romain/dev/js/react-webpack-starter/app/App.jsx
  7:13  error  toto is defined but never used       no-unused-vars
  9:27  error  JSX attributes must use doublequote  react/jsx-quotes

✖ 2 problems (2 errors, 0 warnings)
...
{% endhighlight %}

## The end
Pfiou... enfin terminé, prêt à coder  ! <br>
Vous pouvez retrouver tout le code du projet sur mon github (**webpack.config.js** inclus) : <br> [https://github.com/romaindso/react-webpack-starter][github]

Et pour l'environnement de production ?
A suivre...

Tags: [React, Wepack, ES6, ESLint]

[webpack]: http://webpack.github.io/
[babeljs]: https://babeljs.io/
[features]: https://github.com/lukehoban/es6features
[webpack-dev]: http://webpack.github.io/docs/webpack-dev-server.html
[localhost]: http://localhost:8080/
[eslint]: http://eslint.org/
[rules-eslint]: http://eslint.org/docs/rules
[github]: https://github.com/romaindso/react-webpack-starter
