---
title: Fiche 4
description: Qualité frontend.
permalink: posts/{{ title | slug }}/index.html
date: '2026-02-19'
tags: [fiche, react, quality]
---

# Comment gérer la qualité du front-end ?

La qualité du frontend est essentielle pour offrir une expérience utilisateur optimale et maintenir la cohérence et la maintenabilité du code. Ce tutoriel va explorer quelques pratiques clés pour gérer la qualité du frontend :

1. **Librairie UI** : Utiliser une librairie UI comme Material-UI (MUI) pour créer des composants uniformes avec un look professionnel. MUI offre une large gamme de composants réutilisables et personnalisables qui respectent les principes de conception Material Design.
2. **Style guide de Google** : Utiliser le style guide de Google pour les conventions de code et de nommage. Cela permet de maintenir un code propre, lisible et cohérent à travers toute l'équipe de développement.
3. **Architecture modulaire** du code : Adopter une architecture modulaire pour le code frontend. Utiliser React Context pour gérer l'état de l'application de manière découplée de la présentation. Cela permet de rendre le code plus maintenable et de faciliter les tests et les mises à jour.
4. __Tests unitaires__ avec Vitest : Écrire des tests unitaires pour vérifier le bon fonctionnement des composants et des fonctions. Utiliser Vitest, un framework de test rapide et léger, pour exécuter les tests unitaires.


---

# 1. Librairie UI

Il existe de nombreuses librairies de composants React qui permettent de rendre plus facile et plus rapide la création de UI : Material UI, Ant Design, Chakra UI, ...

Pour ce tutoriel optionnel, nous allons utiliser **Material UI**, une librairie de composants React très populaire qui permet de créer des applications web modernes et réactives.

Toute la documentation de cette librairie est disponible : https://mui.com/material-ui/

## Mise en place de Material UI </InternalPageTitle>

### a) Installation de Material UI

Pour ce tutoriel, veuillez télécharger ce starter : <LinkFile name="/files/ui-library-starter.zip" target="_blank" download>ui-library-starter</LinkFile>

Vous trouvez aussi ce starter dans le repository de ce cours : [ui-library-starter](https://github.com/e-vinci/cae-theory-demos/tree/main/ui-library-starter).

Veuillez renommer votre répertoire et nom de projet en  `ui-library`.


Commencez par installer les librairies de bases nécessaires :
```sh
npm install @mui/material @emotion/react @emotion/styled @fontsource/roboto @mui/icons-material
```

### b) Utilisation de Material UI

Par défaut, Material UI utilise le font `Roboto` qu'il faut installer. Pour utiliser un font, il faut ensuite faire un `import`, par exemple dans votre script d'entrée de votre application `/src/main.tsx` :
```tsx
import "@fontsource/roboto/700.css";
```

### c) Reset global du CSS

Il est de bonne pratique de normaliser tous les composants HTML en faisant appel à `CssBaseline`.

Veuillez mettre à jour `/src/main.tsx` ainsi :

```tsx {4,10} showLineNumbers
import React from "react";
import ReactDOM from "react-dom/client";
import "@fontsource/roboto/700.css";
import CssBaseline from "@mui/material/CssBaseline";
import App from "./components/App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <CssBaseline /> {/* Global CSS reset from Material-UI */}
    <App />
  </React.StrictMode>
);
```

## Principe général de fonctionnement de MUI

`MUI` met à disposition beaucoup de composants qui permettent de créer des UI en utilisant les règles de `Material Design` comme référence.

Tous les composants peuvent être découverts ici : https://mui.com/material-ui/all-components/

Les composants peuvent être taillés sur mesure selon différentes stratégies. En voici les principales :
- Customiser un seul élément d'un composant `MUI` via la prop `sx` (les valeurs sont un superset de CSS) ou via la prop `className` (pour utiliser des classes CSS personnelles);
- Créer un composant réutilisable à partir d'un composant `MUI` et de l'utilitaire `styled` ;
- Faire une surcharge d'un composant `MUI` via un `theme` ;
- Faire une surcharge globale du CSS de certains éléments HTML en utilisant le composant `GlobalStyles`.

Dans ce tutorial, nous allons explorer la première option uniquement à l'aide de `sx`. N'hésitez pas à en découvrir plus par vous-même via : https://mui.com/material-ui/customization/how-to-customize/

Il existe des composants de layout qui permettent d'agencer d'autres composants horizontalement ou verticalement, principalement :
- `Box` : Un composant qui sert de conteneur flexible pour appliquer des marges, des paddings, des alignements et d'autres styles CSS aux enfants.
- `Container` : Un composant qui centre et limite la largeur du contenu à une taille prédéfinie pour maintenir des marges cohérentes et une mise en page réactive.
- `Grid` : Un composant pour créer des mises en page en grille réactives, permettant de définir des rangées et des colonnes avec des espacements et des alignements configurables.
- `Stack` : Un composant qui simplifie l'agencement des enfants en les empilant verticalement ou horizontalement avec des espacements uniformes.

Le système de breakpoints de Material UI permet de créer des mises en page réactives, c'est-à-dire en ajustant le rendu des composants en fonction de la taille de l'écran. Material UI propose plusieurs breakpoints par défaut qui correspondent à des largeurs d'écran courantes.

Les breakpoints par défaut de Material UI sont définis comme suit :
- `xs` (extra-small): 0px et plus
- `sm` (small): 600px et plus
- `md` (medium): 900px et plus
- `lg` (large): 1200px et plus
- `xl` (extra-large): 1536px et plus


Par exemple, le composant Grid utilise ces breakpoints pour définir le nombre de colonnes à afficher à différentes largeurs d'écran :

{% raw %}
```tsx
<Grid container spacing={2}>
  <Grid size={{ xs: 12, md: 6 }}>
    <Item>xs=12 md=6</Item>
  </Grid>
 <Grid size={{ xs: 12, md: 6 }}>
    <Item>xs=12 md=6</Item>
  </Grid>
 <Grid size={{ xs: 12, md: 6 }}>
    <Item>xs=12 md=6</Item>
  </Grid>
  <Grid size={{ xs: 12, md: 6 }}>
    <Item>xs=12 md=6</Item>
  </Grid>
</Grid>
```
{% endraw %}

Ce `Grid` permet à un composant d'occuper 6 colonnes sur 12 du viewport quand la largeur du viewport est de 600 et plus pixel. Pour les viewport plus petits, le composant remplit les 12 colonnes disponibles. Cela permet de créer une mise en page réactive qui s'adaptent à la taille de l'écran : soit 2 colonnes sur un écran large, soit 1 colonne sur un écran plus petit.

## Utilisation de base des composants MUI

Pour ce tutoriel, les composants `MUI` qui semblent applicables à l'UI de notre application gérant une pizzeria ont été sélectionnés sur base de la documentation de `MUI`.

Nous vous proposons de mettre à jour les scripts de votre projet sans aucune gestion du style : nous allons donc enlever toutes les références au CSS, et nous n'allons pas encore utiliser le système de `MUI` pour styler les éléments de notre applications.

Nous avons déjà mis à jour `/src/main.tsx`. Continuons donc par la mise à jour de `Main` (dans `/src/components/App/index.tsx`) :
```tsx {5,19,31} showLineNumbers
import Footer from "../Footer";
import Header from "../Header";
import Main from "../Main";
import { useState } from "react";
import { Box } from "@mui/material";

function App() {
  const [actionToBePerformed, setActionToBePerformed] = useState(false);

  const handleHeaderClick = () => {
    setActionToBePerformed(true);
  };

  const clearActionToBePerformed = () => {
    setActionToBePerformed(false);
  };

  return (
    <Box>
      <Header
        title="We love Pizza"
        version={0 + 1}
        handleHeaderClick={handleHeaderClick}
      />
      <Main
        actionToBePerformed={actionToBePerformed}
        clearActionToBePerformed={clearActionToBePerformed}
      />

      <Footer />
    </Box>
  );
}

export default App;
```

Nous avons juste utilisé `Box` pour prendre la place d'une `div` et nous n'utilisons plus `App.css`.

Veuillez ensuite mettre à jour le `Header` (dans `/src/components/Header/index.tsx`) :
```tsx {1,20-21,24-29} showLineNumbers
import { Box, Container, Typography } from "@mui/material";
import { useState } from "react";

interface HeaderProps {
  title: string;
  version: number;
  handleHeaderClick: () => void;
}

const Header = ({ title, handleHeaderClick }: HeaderProps) => {
  const [menuPrinted, setMenuPrinted] = useState(false);

  const handleClick = () => {
    console.log(`value of menuPrinted before click: ${menuPrinted}`);
    setMenuPrinted(!menuPrinted);
    handleHeaderClick();
  };

  return (
    <Box
      component="header"
      onClick={handleClick}
    >
      <Container maxWidth="sm">
        <Typography variant="h1">
          {menuPrinted ? `${title}... and rarely do we hate it!` : title}
        </Typography>
      </Container>
    </Box>
  );
};

export default Header;
```

Là nous utilisons :
- `Box` : pour créer notre future élément `<header>`.
- `Container` : pour créer un container qui centre ses éléments et dont la largeur vaut maximum `sm` (600px et plus).
- `Typography` : pour gérer le texte (qui deviendra un élément `<h1>`).

Nous n'utilisons plus `Header.css`.

Veuillez ensuite mettre à jour le `Main` (dans `/src/components/Main/index.tsx`). Notons que nous souhaitons faire un refactor de l'application afin que le menu des boissons soit affiché sur base d'une collection de données :

{% raw %}
```tsx {8,38-60,76-77,79-83,98} showLineNumbers
import { useState } from "react";
import sound from "../../assets/sounds/Infecticide-11-Pizza-Spinoza.mp3";
import DrinkMenu from "./DrinkMenu";
import PizzaMenu from "./PizzaMenu";
import { NewPizza, Pizza, Drink} from "../../types";
import AddPizza from "./AddPizza";
import AudioPlayer from "./AudioPlayer";
import { Container, Typography } from "@mui/material";

const defaultPizzas: Pizza[] = [
  {
    id: 1,
    title: "4 fromages",
    content: "Gruyère, Sérac, Appenzel, Gorgonzola, Tomates",
  },
  {
    id: 2,
    title: "Vegan",
    content: "Tomates, Courgettes, Oignons, Aubergines, Poivrons",
  },
  {
    id: 3,
    title: "Vegetarian",
    content: "Mozarella, Tomates, Oignons, Poivrons, Champignons, Olives",
  },
  {
    id: 4,
    title: "Alpage",
    content: "Gruyère, Mozarella, Lardons, Tomates",
  },
  {
    id: 5,
    title: "Diable",
    content: "Tomates, Mozarella, Chorizo piquant, Jalapenos",
  },
];

const drinks: Drink[] = [
  {
    title: "Coca-Cola",
    image:
      "https://media.istockphoto.com/id/1289738725/fr/photo/bouteille-en-plastique-de-coke-avec-la-conception-et-le-chapeau-rouges-d%C3%A9tiquette.jpg?s=1024x1024&w=is&k=20&c=HBWfROrGDTIgD6fuvTlUq6SrwWqIC35-gceDSJ8TTP8=",
    volume: "Volume: 33cl",
    price: "2,50 €",
  },
  {
    title: "Pepsi",
    image:
      "https://media.istockphoto.com/id/185268840/fr/photo/bouteille-de-cola-sur-un-fond-blanc.jpg?s=1024x1024&w=is&k=20&c=xdsxwb4bLjzuQbkT_XvVLyBZyW36GD97T1PCW0MZ4vg=",
    volume: "Volume: 33cl",
    price: "2,50 €",
  },
  {
    title: "Eau Minérale",
    image:
      "https://media.istockphoto.com/id/1397515626/fr/photo/verre-deau-gazeuse-%C3%A0-boire-isol%C3%A9.jpg?s=1024x1024&w=is&k=20&c=iEjq6OL86Li4eDG5YGO59d1O3Ga1iMVc_Kj5oeIfAqk=",
    volume: "Volume: 50cl",
    price: "1,50 €",
  },
];

interface MainProps {
  actionToBePerformed: boolean;
  clearActionToBePerformed: () => void;
}

const Main = ({ actionToBePerformed, clearActionToBePerformed }: MainProps) => {
  const [pizzas, setPizzas] = useState(defaultPizzas);

  const addPizza = (newPizza: NewPizza) => {
    const pizzaAdded = { ...newPizza, id: nextPizzaId(pizzas) };
    setPizzas([...pizzas, pizzaAdded]);
  };

  return (
    <Container component="main" sx={{ mt: 8, mb: 2, flex: "1" }} maxWidth="sm">
      <Typography variant="h2" component="h1" gutterBottom>
        My HomePage
      </Typography>
      <Typography variant="h5" component="h2" gutterBottom>
        Because we love JS, you can also click on the header to stop / start the
        music ; )
      </Typography>
      <AudioPlayer
        sound={sound}
        actionToBePerformed={actionToBePerformed}
        clearActionToBePerformed={clearActionToBePerformed}
      />

      <PizzaMenu pizzas={pizzas} />

      <br/>

      <AddPizza addPizza={addPizza} />

      <br/>

      <DrinkMenu title="Notre Menu de Boissons" drinks={drinks} />
    </Container>
  );
};

const nextPizzaId = (pizzas: Pizza[]) => {
  return pizzas.reduce((maxId, pizza) => Math.max(maxId, pizza.id), 0) + 1;
};

export default Main;
```
{% endraw %}

Il n'y a pas de composants `MUI` non rencontrés précédemment. Il n'y a plus de CSS.

A ce stade-ci, il est normal qu'il y ait toujours des erreurs dans votre application. Nous allons les corriger en changeant plusieurs composants.

Voici le nouveau type `Drink` qui doit être ajouté à `/src/types.ts` :
```ts {9-14,16} showLineNumbers
interface Pizza {
  id: number;
  title: string;
  content: string;
}

type NewPizza = Omit<Pizza, "id">;

interface Drink {
  title: string;
  image: string;
  volume: string;
  price: string;
}

export type { Pizza, NewPizza, Drink };
```

Le composant `AudioPlayer` ne doit pas être mis à jour. Le composant `CssBaseline` offre déjà un joli style de base.

Veuillez ensuite mettre à jour `PizzaMenu` (dans `/src/components/Main/PizzaMenu.tsx`) :
```tsx {1-9,18-35} showLineNumbers
import {
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
} from "@mui/material";

import { Pizza } from "../../types";

interface PizzaMenuProps {
  pizzas: Pizza[];
}
const PizzaMenu = ({ pizzas }: PizzaMenuProps) => {
  return (
    <TableContainer component={Paper}>
      <Table>
        <TableHead>
          <TableRow>
            <TableCell>Pizza</TableCell>
            <TableCell>æ©escription</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {pizzas.map((pizza) => (
            <TableRow key={pizza.id}>
              <TableCell>{pizza.title}</TableCell>
              <TableCell>{pizza.content}</TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </TableContainer>
  );
};

export default PizzaMenu;
```

Pour ce composant, nous avons utilisé tous les composants `MUI` permettant de créer une table HTML : https://mui.com/material-ui/react-table/#basic-table.


Pour le menu des boissons, nous avons créé un tout nouveau design pour être basé sur des données plutôt que des composants enfants (via `children`).  
Veuillez mettre à jour `DrinkMenu` (dans `/src/components/Main/DrinkMenu.tsx`) :
{% raw %}
```tsx {1-9,11-14,16-49} showLineNumbers
import {
  Container,
  Card,
  CardMedia,
  CardContent,
  Typography,
  Grid2,
} from "@mui/material";
import { Drink } from "../../types";

interface DrinkMenuProps {
  title: string;
  drinks: Drink[];
}

const DrinkMenu = ({ title, drinks }: DrinkMenuProps) => {
  return (
    <Container>
      <Typography variant="h4" gutterBottom>
        {title}
      </Typography>
      <Grid2 container spacing={3}>
        {drinks.map((drink, index) => (
          <Grid2 size={{ xs: 12, sm: 6 }} key={index}>
            <Card>
              <CardMedia
                component="img"
                image={drink.image}
                alt={drink.title}
                style={{ objectFit: "contain", height: "200px" }} // Ensure image is fully visible
              />
              <CardContent>
                <Typography gutterBottom variant="h5" component="div">
                  {drink.title}
                </Typography>
                <Typography variant="body2" color="textSecondary" component="p">
                  {drink.volume}
                </Typography>
                <Typography variant="body2" color="textSecondary" component="p">
                  Prix: {drink.price}
                </Typography>
              </CardContent>
            </Card>
          </Grid2>
        ))}
      </Grid2>
    </Container>
  );
};

export default DrinkMenu;
```
{% endraw %}

Nous avons utilisé le composant `Card` du `MUI` pour l'UI de chaque boisson : https://mui.com/material-ui/react-card/.

Veuillez ensuite mettre à jour `AddPizza` (dans `/src/components/Main/AddPizza.tsx`) :
{% raw %}
```tsx {4,32-64} showLineNumbers
import { useState, SyntheticEvent } from "react";

import { NewPizza } from "../../types";
import { Box, Button, TextField } from "@mui/material";

interface AddPizzaProps {
  addPizza: (pizza: NewPizza) => void;
}

const AddPizza = ({ addPizza }: AddPizzaProps) => {
  const [pizza, setPizza] = useState("");
  const [description, setDescription] = useState("");

  const handleSubmit = (e: SyntheticEvent) => {
    e.preventDefault();
    addPizza({ title: pizza, content: description });
  };

  const handlePizzaChange = (e: SyntheticEvent) => {
    const pizzaInput = e.target as HTMLInputElement;
    console.log("change in pizzaInput:", pizzaInput.value);
    setPizza(pizzaInput.value);
  };

  const handleDescriptionChange = (e: SyntheticEvent) => {
    const descriptionInput = e.target as HTMLInputElement;
    console.log("change in descriptionInput:", descriptionInput.value);
    setDescription(descriptionInput.value);
  };

  return (
    <Box>
      <form onSubmit={handleSubmit}>
        <Box sx={{ marginBottom: 2 }}>
          <TextField
            fullWidth
            id="pizza"
            name="pizza"
            label="Pizza"
            variant="outlined"
            value={pizza}
            onChange={handlePizzaChange}
            required
            color="primary"
          />
        </Box>
        <Box sx={{ marginBottom: 2 }}>
          <TextField
            fullWidth
            id="description"
            name="description"
            label="Description"
            variant="outlined"
            value={description}
            onChange={handleDescriptionChange}
            required
            color="primary"
          />
        </Box>
        <Button type="submit" variant="contained" color="primary">
          Ajouter
        </Button>
      </form>
    </Box>
  );
};

export default AddPizza;
```
{% endraw %}

Nous avons utilisé `TextField` afin de gérer les 2 inputs de notre formulaire.

Il ne reste plus qu'à mettre à jour `Footer` (dans `/src/components/Footer/index.tsx`) :
```tsx {1,7-20} showLineNumbers
import { Box, Container, Typography } from "@mui/material";
import logo from "../../assets/images/js-logo.png";
import { Copyright } from "@mui/icons-material";

const Footer = () => {
  return (
    <Box component="footer" color="">
      <Container maxWidth="sm">
        <Box>
          <Typography variant="body2">But we also love JS</Typography>
          <Typography>
            <Copyright />
            myAmazingPizzeria
          </Typography>
        </Box>
        <Box>
          <img src={logo} alt="" width={50} />
        </Box>
      </Container>
    </Box>
  );
};

export default Footer;
```

Nous avons utilisé l'icône `Copyright` pour le `Footer`.

Veuillez exécutez l'application !  
Le résultat est fort intéressant : l'interface est très épurée, avec un look assez professionnel. Mais ça manque de style !

## Exemple d'utilisation de la prop sx

Notre application possède déjà un thème par défaut, même si nous ne l'utilisons actuellement pas vraiment.

Nous allons donc maintenant styler de manière individuelle chacun des éléments `MUI` à l'aide de la prop `sx`. En fait, vous allez faire du CSS très ciblé, sans devoir créer de classes.

Voici la mise à jour du composant `App` afin d'ajouter la photo de background et de s'assurer que l'application prendra au minimum une hauteur de 100% du viewport (pour avoir un footer qui sera toujours en bas de page) :
{% raw %}
```tsx {1,15-21} showLineNumbers
import pizza from "../../assets/images/pizza.jpg";
// Other imports...
function App() {
  const [actionToBePerformed, setActionToBePerformed] = useState(false);

  const handleHeaderClick = () => {
    setActionToBePerformed(true);
  };

  const clearActionToBePerformed = () => {
    setActionToBePerformed(false);
  };

  return (
    <Box sx={{
      display: 'flex',
      flexDirection: 'column',
      height: '100%',
      backgroundImage: `url(${pizza})`,
      backgroundSize: 'cover',
    }}>
      
      

      <Header
        title="We love Pizza"
        version={0 + 1}
        handleHeaderClick={handleHeaderClick}
      />
      <Main
        actionToBePerformed={actionToBePerformed}
        clearActionToBePerformed={clearActionToBePerformed}
      />
      
      <Footer />
    </Box>
  );
}
```


Voici la mise à jour du composant `Header` afin d'obtenir la couleur du thème :
```tsx {1,11,23-30} showLineNumbers
import { Box, Container, Typography, useTheme } from "@mui/material";
import { useState } from "react";

interface HeaderProps {
  title: string;
  version: number;
  handleHeaderClick: () => void;
}

const Header = ({ title, handleHeaderClick }: HeaderProps) => {
  const theme = useTheme();
  const [menuPrinted, setMenuPrinted] = useState(false);

  const handleClick = () => {
    console.log(`value of menuPrinted before click: ${menuPrinted}`);
    setMenuPrinted(!menuPrinted);
    handleHeaderClick();
  };

  return (
    <Box
      component="header"
      sx={{
        px: 2,
        backgroundColor:
          theme.palette.mode === "light"
            ? theme.palette.primary.light
            : theme.palette.primary.dark,
        color: (theme) => theme.palette.primary.contrastText,
      }}
      onClick={handleClick}
    >
      <Container maxWidth="sm">
        <Typography variant="h1">
          {menuPrinted ? `${title}... and rarely do we hate it!` : title}
        </Typography>
      </Container>
    </Box>
  );
};

export default Header;
```

Il y a différents moyens d'obtenir le thème, mais nous trouvons que le hook `useTheme` est le plus simple. Ici, le thème par défaut de `MUI` sera utilisé pour la couleur `primary` (une sorte de bleu).

Pour information, nous avons déjà stylé le composant `Main` afin qu'il prenne tout l'espace disponible (`flex:"1"`) pour assurer que le `Footer` soit toujours tout en bas de la page :
```tsx {76} showLineNumbers
import { useState } from "react";
import sound from "../../assets/sounds/Infecticide-11-Pizza-Spinoza.mp3";
import DrinkMenu from "./DrinkMenu";
import PizzaMenu from "./PizzaMenu";
import { NewPizza, Pizza, Drink} from "../../types";
import AddPizza from "./AddPizza";
import AudioPlayer from "./AudioPlayer";
import { Container, Typography } from "@mui/material";

const defaultPizzas: Pizza[] = [
  {
    id: 1,
    title: "4 fromages",
    content: "Gruyère, Sérac, Appenzel, Gorgonzola, Tomates",
  },
  {
    id: 2,
    title: "Vegan",
    content: "Tomates, Courgettes, Oignons, Aubergines, Poivrons",
  },
  {
    id: 3,
    title: "Vegetarian",
    content: "Mozarella, Tomates, Oignons, Poivrons, Champignons, Olives",
  },
  {
    id: 4,
    title: "Alpage",
    content: "Gruyère, Mozarella, Lardons, Tomates",
  },
  {
    id: 5,
    title: "Diable",
    content: "Tomates, Mozarella, Chorizo piquant, Jalapenos",
  },
];

const drinks: Drink[] = [
  {
    title: "Coca-Cola",
    image:
      "https://media.istockphoto.com/id/1289738725/fr/photo/bouteille-en-plastique-de-coke-avec-la-conception-et-le-chapeau-rouges-d%C3%A9tiquette.jpg?s=1024x1024&w=is&k=20&c=HBWfROrGDTIgD6fuvTlUq6SrwWqIC35-gceDSJ8TTP8=",
    volume: "Volume: 33cl",
    price: "2,50 €",
  },
  {
    title: "Pepsi",
    image:
      "https://media.istockphoto.com/id/185268840/fr/photo/bouteille-de-cola-sur-un-fond-blanc.jpg?s=1024x1024&w=is&k=20&c=xdsxwb4bLjzuQbkT_XvVLyBZyW36GD97T1PCW0MZ4vg=",
    volume: "Volume: 33cl",
    price: "2,50 €",
  },
  {
    title: "Eau Minérale",
    image:
      "https://media.istockphoto.com/id/1397515626/fr/photo/verre-deau-gazeuse-%C3%A0-boire-isol%C3%A9.jpg?s=1024x1024&w=is&k=20&c=iEjq6OL86Li4eDG5YGO59d1O3Ga1iMVc_Kj5oeIfAqk=",
    volume: "Volume: 50cl",
    price: "1,50 €",
  },
];

interface MainProps {
  actionToBePerformed: boolean;
  clearActionToBePerformed: () => void;
}

const Main = ({ actionToBePerformed, clearActionToBePerformed }: MainProps) => {
  const [pizzas, setPizzas] = useState(defaultPizzas);

  const addPizza = (newPizza: NewPizza) => {
    const pizzaAdded = { ...newPizza, id: nextPizzaId(pizzas) };
    setPizzas([...pizzas, pizzaAdded]);
  };

  return (
    <Container component="main" sx={{ mt: 8, mb: 2, flex: "1" }} maxWidth="sm">
      <Typography variant="h2" component="h1" gutterBottom>
        My HomePage
      </Typography>
      <Typography variant="h5" component="h2" gutterBottom>
        Because we love JS, you can also click on the header to stop / start the
        music ; )
      </Typography>
      <AudioPlayer
        sound={sound}
        actionToBePerformed={actionToBePerformed}
        clearActionToBePerformed={clearActionToBePerformed}
      />

      <PizzaMenu pizzas={pizzas} />

      <br/>

      <AddPizza addPizza={addPizza} />

      <br/>

      <DrinkMenu title="Notre Menu de Boissons" drinks={drinks} />
    </Container>
  );
};

const nextPizzaId = (pizzas: Pizza[]) => {
  return pizzas.reduce((maxId, pizza) => Math.max(maxId, pizza.id), 0) + 1;
};

export default Main;
```

Nous allons maintenant mettre à jour le `PizzaMenu` afin d'ajouter des couleurs de la palette du thème par défaut à la table HTML :
```tsx {9,18,22-36} showLineNumbers
import {
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  useTheme,
} from "@mui/material";

import { Pizza } from "../../types";

interface PizzaMenuProps {
  pizzas: Pizza[];
}
const PizzaMenu = ({ pizzas }: PizzaMenuProps) => {
  const theme = useTheme();
  return (
    <TableContainer component={Paper}>
      <Table
        sx={{
          minWidth: 500,
          "& .MuiTableCell-head": {
            backgroundColor: theme.palette.primary.dark,
            color: theme.palette.primary.contrastText,
            fontWeight: "bold",
          },
          "& .MuiTableCell-body": {
            backgroundColor: theme.palette.primary.light,
            color: "white",
          },
          "& .MuiTableCell-root": {
            border: `1px solid ${theme.palette.secondary.main} `,
          },
        }}
      >
        <TableHead>
          <TableRow>
            <TableCell>Pizza</TableCell>
            <TableCell>Description</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {pizzas.map((pizza) => (
            <TableRow key={pizza.id}>
              <TableCell>{pizza.title}</TableCell>
              <TableCell>{pizza.content}</TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </TableContainer>
  );
};

export default PizzaMenu;
```

Nous utilisons ici la notion de `&` qui vient du monde CSS / SASS permettant de cibler des classes ou des pseudos-classes imbriquées à l'intérieur d'un sélecteur parent spécifié. Cela facilite la création de règles CSS spécifiques à des contextes particuliers sans avoir à répéter le sélecteur parent complet.  
Si vous souhaitez en savoir plus sur cette pratique : https://mui.com/material-ui/customization/how-to-customize/#overriding-nested-component-styles

Voici comment nous stylons le composant `DrinkMenu` :
```tsx {8,18,24-28,34} showLineNumbers
import {
  Container,
  Card,
  CardMedia,
  CardContent,
  Typography,
  Grid2,
  useTheme,
} from "@mui/material";
import { Drink } from "../../types";

interface DrinkMenuProps {
  title: string;
  drinks: Drink[];
}

const DrinkMenu = ({ title, drinks }: DrinkMenuProps) => {
  const theme = useTheme();
  return (
    <Container>
      <Typography
        variant="h4"
        gutterBottom
        sx={{
          color: theme.palette.primary.contrastText,
          textAlign: "center",
          marginTop: 2,
        }}
      >
        {title}
      </Typography>
      <Grid2 container spacing={3}>
        {drinks.map((drink, index) => (
          <Grid2 size={{ xs: 12, sm: 6 }} key={index}>
            <Card>
              <CardMedia
                component="img"
                image={drink.image}
                alt={drink.title}
                style={{ objectFit: "contain", height: "200px" }} // Ensure image is fully visible
              />
              <CardContent>
                <Typography gutterBottom variant="h5" component="div">
                  {drink.title}
                </Typography>
                <Typography variant="body2" color="textSecondary" component="p">
                  {drink.volume}
                </Typography>
                <Typography variant="body2" color="textSecondary" component="p">
                  Prix: {drink.price}
                </Typography>
              </CardContent>
            </Card>
          </Grid2>
        ))}
      </Grid2>
    </Container>
  );
};

export default DrinkMenu;
```

Voici comment mettre à jour le style du formulaire au sein de `AddPizza` :
```tsx {34-40,43,54-56,59,70-72} showLineNumbers
import { useState, SyntheticEvent } from "react";

import { NewPizza } from "../../types";
import { Box, Button, TextField, useTheme } from "@mui/material";

interface AddPizzaProps {
  addPizza: (pizza: NewPizza) => void;
}

const AddPizza = ({ addPizza }: AddPizzaProps) => {
  const theme = useTheme();
  const [pizza, setPizza] = useState("");
  const [description, setDescription] = useState("");

  const handleSubmit = (e: SyntheticEvent) => {
    e.preventDefault();
    addPizza({ title: pizza, content: description });
  };

  const handlePizzaChange = (e: SyntheticEvent) => {
    const pizzaInput = e.target as HTMLInputElement;
    console.log("change in pizzaInput:", pizzaInput.value);
    setPizza(pizzaInput.value);
  };

  const handleDescriptionChange = (e: SyntheticEvent) => {
    const descriptionInput = e.target as HTMLInputElement;
    console.log("change in descriptionInput:", descriptionInput.value);
    setDescription(descriptionInput.value);
  };

  return (
    <Box
      sx={{
        marginTop: 2,
        padding: 3,
        backgroundColor: "secondary.light",
        borderRadius: 4,
        boxShadow: 2,
      }}
    >
      <form onSubmit={handleSubmit}>
        <Box sx={{ marginBottom: 2 }}>
          <TextField
            fullWidth
            id="pizza"
            name="pizza"
            label="Pizza"
            variant="outlined"
            value={pizza}
            onChange={handlePizzaChange}
            required
            color="primary"
            sx={{
              input: { color: theme.palette.secondary.contrastText },
            }}
          />
        </Box>
        <Box sx={{ marginBottom: 2 }}>
          <TextField
            fullWidth
            id="description"
            name="description"
            label="Description"
            variant="outlined"
            value={description}
            onChange={handleDescriptionChange}
            required
            color="primary"
            sx={{
              input: { color: theme.palette.secondary.contrastText },
            }}
          />
        </Box>
        <Button type="submit" variant="contained" color="primary">
          Ajouter
        </Button>
      </form>
    </Box>
  );
};

export default AddPizza;
```

Nous pouvons maintenant mettre à jour le `Footer` :
```tsx {1,6,11-17,21-25,33} showLineNumbers
import { Box, Container, Typography, useTheme } from "@mui/material";
import logo from "../../assets/images/js-logo.png";
import { Copyright } from "@mui/icons-material";

const Footer = () => {
  const theme = useTheme();

  return (
    <Box
      component="footer"
      sx={{
        py: 3,
        backgroundColor:
          theme.palette.mode === "light"
            ? theme.palette.secondary.light
            : theme.palette.secondary.dark,
      }}
    >
      <Container maxWidth="sm">
        <Box
          sx={{
            display: "inline-block",
            paddingRight: 2,
            color: theme.palette.secondary.contrastText,
          }}
        >
          <Typography variant="body1">But we also love JS</Typography>
          <Typography>
            <Copyright />
            myAmazingPizzeria
          </Typography>
        </Box>
        <Box sx={{ display: "inline-block" }}>
          <img src={logo} alt="" width={50} />
        </Box>
      </Container>
    </Box>
  );
};

export default Footer;
```

Nous avons maintenant une application qui commence à être bien stylée !

Il y a un souci qui est visible sur toutes les applications offertes par les templates de projet de `MUI`. Il y a toujours un espace, une sorte de marge en bas de page, après notre `Footer`.

Pour résoudre ce souci, qui ne semble malheureusement pas documenté sur le Web, il vous est proposé d'ajouter une seule feuille de style à votre application, au niveau de `/src/main.tsx`, veuillez importer `/src/index.css` contenant ce code :
```css
div#root {
  width: 100%;
  display: inline-block; /* avoid margins to collapse to avoid vertical scrollbar */
}
```
{% endraw %}

Wow, nous avons quelque chose de fonctionnel !

## Création de son propre thème

Pour ce tutoriel, nous vous proposons de créer la palette de couleurs la plus simple pour donner les couleurs primaires et secondaires que nous souhaitons pour un site d'une pizzeria.

Il existe des outils très intéressants pour créer ses thèmes et palettes de couleurs. Vous trouvez ceux-ci ici :
- [Material palette generator](https://m2.material.io/inline-tools/color/)
- [mui-theme-creator](https://zenoo.github.io/mui-theme-creator/)
- Autres outils pour générer ou découvrir des palettes : https://mui.com/material-ui/customization/color/

Veuillez créer un fichier pour y ajouter la définition d'un nouveau thème dans `/src/themes.ts` :
```ts
import { createTheme } from "@mui/material/styles";

const theme = createTheme({
  palette: {
    primary: {
      main: "#f0483b",
    },
    secondary: {
      main: "#3bf048",
    },
  },
});

export default theme;
```

Et pour utiliser ce nouveau thème, nous devons créer un provider qui va "injecter" ce thème dans l'arbre de tous les composants React. Pour ce faire, veuillez mettre à jour `/src/main.tsx` :

```tsx {5,13,16} showLineNumbers
import React from "react";
import ReactDOM from "react-dom/client";
import "@fontsource/roboto/700.css";
import CssBaseline from "@mui/material/CssBaseline";
import { ThemeProvider } from "@mui/material/styles";
import theme from "./themes";

import App from "./components/App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <ThemeProvider theme={theme}>
      <CssBaseline /> {/* Global CSS reset from Material-UI */}
      <App />
    </ThemeProvider>
  </React.StrictMode>
);
```

Si nécessaire, vous pouvez trouver le code associé à ce tutoriel ici : [ui-library](https://github.com/e-vinci/cae-theory-demos/tree/main/ui-library).
Ceci termine la première partie du tutoriel consacrée à l'utilisation d'une Librairie UI.

---


# 2. Style guide
Le style guide de Google est une référence pour les conventions de code et de nommage en TypeScript. Il fournit des recommandations pour écrire un code propre, lisible et cohérent. En suivant les conventions du style guide de Google, vous pouvez améliorer la qualité de votre code et faciliter la collaboration au sein de l'équipe de développement.

Il est disponible en ligne à l'adresse suivante : https://google.github.io/styleguide/tsguide.html.

Le style guide de Google pour TypeScript couvre plusieurs aspects du développement, tels que la syntaxe, le formatage, les conventions de nommage, les bonnes pratiques et les recommandations pour la documentation du code. Il est conçu pour être utilisé avec l'outil de formatage de code Prettier, qui permet de formater automatiquement le code selon les conventions du style guide.

Il est aussi possible d'utiliser le linter ESLint avec le plugin `@typescript-eslint/eslint-plugin` pour vérifier le respect des conventions du style guide de Google. Le plugin `@typescript-eslint/eslint-plugin` fournit des règles spécifiques pour TypeScript qui permettent de détecter les erreurs et les problèmes potentiels dans le code.

## Résumé du style guide

Voici un minuscule résumé de quelques points du style guide de Google pour TypeScript afin de vous donner une idée des conventions de codage recommandées :

- **Nommage des variables et des fonctions** :
    - Utiliser le **c**amel**C**ase pour les noms de variables et de fonctions. Par exemple: `myVariable`, `myFunction`.
    - Utiliser **P**ascal**C**ase pour les noms de composants fonctionnels et d'interfaces. Par exemple : `LoginPage`, `LoginPageProps` (interface).
    - ...

- **Types et interfaces** :
    - Préférer les interfaces aux types pour définir des objets et des fonctions.
    - Utiliser des types pour les unions et les intersections.
    - Toujours annoter les types des paramètres et des retours de fonctions.

- **Imports et exports** :
    - Utiliser des imports et exports explicites plutôt que des imports et exports par défaut.
    - Grouper les imports par type : bibliothèques tierces, modules internes, styles, etc.

- **Formatage du code** :
    - Utiliser des outils de formatage automatique comme `Prettier` pour assurer la cohérence du code.
    - Respecter une largeur de ligne maximale de `80` caractères.
    - Utiliser des espaces pour l'indentation, avec une taille de tabulation de `2` espaces.

Plutôt que de devoir apprendre les règles, le plus simple est de mettre en place un linter et un formatter pour vérifier et formater automatiquement le code selon les conventions du style guide de Google.

## Mise en place du linter et du formatter

### Installation des dépendances

Pour ce tutoriel, veuillez créer un nouveau projet nommé `style-guide` sur base du tutoriel précédent.  
Si nécessaire, vous pouvez trouver le code associé au tutoriel précédent ici : [ui-library](https://github.com/e-vinci/cae-theory-demos/tree/main/ui-library).

Veuillez installer les dépendances suivantes :
```sh
npm install -D eslint prettier eslint-config-google eslint-plugin-prettier eslint-config-prettier
```

Veuillez ensuite mettre à jour le fichier de configuration de eslint `.eslintrc.cjs` avec ces ajouts :

```js {8-9,15-16} showLineNumbers
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended",
    "google",
     "plugin:prettier/recommended",
  ],
  ignorePatterns: ["dist", ".eslintrc.cjs"],
  parser: "@typescript-eslint/parser",
  plugins: ["react-refresh"],
  rules: {
    "prettier/prettier": "error",
    "require-jsdoc": "off",
    "react-refresh/only-export-components": [
      "warn",
      { allowConstantExport: true },
    ],
  },
};
```

**`"google"`** est un preset de règles d'ESLint qui suit les conventions de codage du style guide de Google.

**`"plugin:prettier/recommended"`** est un plugin ESLint qui permet d'utiliser Prettier pour formater le code.

**`"prettier/prettier": "error"`** est une règle qui indique à ESLint d'afficher une erreur si le code n'est pas formaté correctement.

Veuillez ensuite créer le fichier de configuration du formatter `.prettierrc` avec le contenu suivant :

```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 80
}
```

**`"singleQuote": true`** permet d'utiliser des quotes simples pour les chaînes de caractères. Quand vous utiliserez Prettier, il remplacera les quotes doubles par des quotes simples.

**`"trailingComma": "all"`** permet d'ajouter une virgule à la fin des objets et des tableaux, même si c'est le dernier élément.

**`"printWidth": 80`** permet de limiter la largeur de ligne à 80 caractères.

Veuillez ensuite mettre à `package.json` pour ajouter le script "format" (et vous assurer que le script "lint" est bien présent) :

```json {4,6}
"scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview",
    "format": "prettier --write ."
  },
```

Ici on n'autorise aucun warning avec `--max-warnings 0`. Cela permet de forcer le respect des règles de formatage.

## Utilisation du linter et du formatter

Vous pouvez voir tous les fichiers qui ne respectent pas les règles de formatage :

```bash
npm run lint
```
Vous devriez voir dans votre projet qu'il y a environ une centaine d'erreurs.

Vous pouvez réparer les erreurs de formatage avec prettier :

```bash
npm run format
```

Si vous avez installé l'extension Prettier dans VSCode, vous pouvez formater le fichier en appuyant sur `SHIFT + ALT + F`.

Il est possible qu'il reste encore une ou l'autre erreur à régler manuellement.
Par exemple, dans le fichier `vite-env.d.ts`, vous pouvez ajouter un commentaire pour désactiver le linter :

```ts
// eslint-disable-next-line spaced-comment
/// <reference types="vite/client" />
```

Veuillez ensuite relancer la commande `npm run lint` pour vérifier qu'il n'y a plus d'erreurs.

## Que se passe-t-il si on dépasse les 80 caractères par ligne ?

Dans le composant `Header`, veuillez ajouter un texte long pour dépasser les 80 caractères par ligne :

```tsx
console.log(`value of menuPrinted before clickssssssssssssssssssssss : ${menuPrinted}`);
```

Lancez la commande `npm run lint` pour voir le warning.

Vous pouvez régler le problème de plusieurs façons :
- Vous pouvez utiliser l'option `Quick Fix` de VSCode pour formater le fichier.
- Si vous avez installé l'extension Prettier dans VSCode, vous pouvez formater le fichier en appuyant sur `SHIFT + ALT + F`.
- Vous pouvez aussi utiliser la commande `npm run format` pour formater tous les fichiers.

Une fois le problème résolu, relancez la commande `npm run lint` pour vérifier qu'il n'y a plus de warning.

Si nécessaire, vous pouvez trouver le code associé à ce tutoriel ici : [style-guide](https://github.com/e-vinci/cae-theory-demos/tree/main/style-guide).

---

# 3. Architecture modulaire
Afin de garder un code maintenable, et pour simplifier les tests unitaires, il est important de découpler la logique métier de la présentation. Pour cela, React propose un mécanisme appelé Context qui permet de gérer l'état de l'application de manière centralisée et de le partager entre les composants sans passer par les props.

## Fonctionnement de React Context

Afin d'utiliser un contexte nous allons avoir besoin de 3 morceaux liés les uns aux autres :

- le **contexte** en lui-même, fabriqué par React grâce à l'appel : **`React.createContext()`**
- un **`ProviderWrapper`** : un composant très similaire au composant **`App`** (ou **`Boot`**, ou **`AppLoader`**) qui contient tout le state : sa fonction est d'encapsuler le **Provider** brut et définir un state grâce à **`useState`** ainsi que des fonctions qui permettent de manipuler ce state.
  La valeur de son **`return`** sera le **provider** brut à qui il aura passé dans la prop "**`value`**" les différentes valeurs et fonction que nous souhaitons exposer.
- des **consumers** : différents composants qui feront usage du hook **`useContext`**. L'usage de ce hook donnera accès aux valeurs exposées par le **`ProviderWrapper`** le plus proche (en remontant l'arbre des composants). Si aucun n'est trouvé, le hook retournera le **`defaultContext`**.

## Exemple de React Context

Pour ce tutoriel, veuillez télécharger ce starter : <LinkFile name="/files/context-starter.zip" target="_blank" download>context-starter</LinkFile>

Vous trouvez aussi ce starter dans le repository de ce cours : [context-starter](https://github.com/e-vinci/cae-theory-demos/tree/main/context-starter).

Veuillez renommer votre répertoire en  `context`.  
Ce répertoire contient une API à installer & démarrer au sein de `auths` et une application React à installer & démarrer au sein de `front`.

Dans le projet `front`, il y a déjà un `outletContext` qui gère tant les données associées à l'utilisateur connecté que les pizzas et les boissons.

Nous allons nous éloigner de cet `outletContext` et créer un `UserContext` qui gérera l'utilisateur connecté.

Au niveau des types, le `PizzeriaContext` ne doit plus contenir les informations de l'utilisateur (on efface donc loginUser & registerUser).

Nous allons ajouter un type (`UserContextType`) qui contiendra un utilisateur (ou **`undefined`** s'il n'est pas connecté) et des fonctions pour le manipuler.

Voici donc le fichier **`src/types.ts`** que vous devez mettre à jour :
```tsx {16-24,26-31,53} showLineNumbers
interface Pizza {
  id: number;
  title: string;
  content: string;
}

type NewPizza = Omit<Pizza, 'id'>;

interface Drink {
  title: string;
  image: string;
  volume: string;
  price: string;
}

interface PizzeriaContext {
  pizzas: Pizza[];
  setPizzas: (pizzas: Pizza[]) => void;
  actionToBePerformed: boolean;
  setActionToBePerformed: (actionToBePerformed: boolean) => void;
  clearActionToBePerformed: () => void;
  drinks: Drink[];
  addPizza: (newPizza: NewPizza) => Promise<void>;
}

interface UserContextType {
  authenticatedUser: MaybeAuthenticatedUser;
  registerUser: (newUser: User) => Promise<void>;
  loginUser: (user: User) => Promise<void>;
  clearUser: () => void;
}

interface User {
  username: string;
  password: string;
}

interface AuthenticatedUser {
  username: string;
  token: string;
}

type MaybeAuthenticatedUser = AuthenticatedUser | undefined;

export type {
  Pizza,
  NewPizza,
  Drink,
  PizzeriaContext,
  User,
  AuthenticatedUser,
  MaybeAuthenticatedUser,
  UserContextType,
};
```

Ensuite, nous allons créer un nouveau fichier **`src/contexts/UserContext.tsx`** afin de définir notre contexte et notre **`ProviderWrapper`** :
```tsx showLineNumbers
import { createContext, useState, ReactNode } from 'react';
import {
  MaybeAuthenticatedUser,
  UserContextType,
  User,
  AuthenticatedUser,
} from '../types';

import {
  clearAuthenticatedUser,
  storeAuthenticatedUser,
} from '../utils/session';

const defaultUserContext: UserContextType = {
  authenticatedUser: undefined,
  registerUser: async () => {},
  loginUser: async () => {},
  clearUser: () => {},
};

const UserContext = createContext<UserContextType>(defaultUserContext);

const UserContextProvider = ({ children }: { children: ReactNode }) => {
  const [authenticatedUser, setAuthenticatedUser] =
    useState<MaybeAuthenticatedUser>(undefined);

  const registerUser = async (newUser: User) => {
    try {
      const options = {
        method: 'POST',
        body: JSON.stringify(newUser),
        headers: {
          'Content-Type': 'application/json',
        },
      };

      const response = await fetch('/api/auths/register', options);

      if (!response.ok)
        throw new Error(
          `fetch error : ${response.status} : ${response.statusText}`,
        );

      const createdUser: AuthenticatedUser = await response.json();

      setAuthenticatedUser(createdUser);
      storeAuthenticatedUser(createdUser);
      console.log('createdUser: ', createdUser);
    } catch (err) {
      console.error('registerUser::error: ', err);
      throw err;
    }
  };

  const loginUser = async (user: User) => {
    try {
      const options = {
        method: 'POST',
        body: JSON.stringify(user),
        headers: {
          'Content-Type': 'application/json',
        },
      };

      const response = await fetch('/api/auths/login', options);

      if (!response.ok)
        throw new Error(
          `fetch error : ${response.status} : ${response.statusText}`,
        );

      const authenticatedUser: AuthenticatedUser = await response.json();
      console.log('authenticatedUser: ', authenticatedUser);

      setAuthenticatedUser(authenticatedUser);
      storeAuthenticatedUser(authenticatedUser);
    } catch (err) {
      console.error('loginUser::error: ', err);
      throw err;
    }
  };

  const clearUser = () => {
    clearAuthenticatedUser();
    setAuthenticatedUser(undefined);
  };

  const myContext: UserContextType = {
    authenticatedUser,
    registerUser,
    loginUser,
    clearUser,
  };

  return (
    <UserContext.Provider value={myContext}>{children}</UserContext.Provider>
  );
};

export { UserContext, UserContextProvider };
```

Il serait probablement plus propre de séparer les fonctions de fetch dans un fichier **`src/services/auth.ts`** ; nous ne le ferons pas pour ce tutoriel-ci.


Ensuite, nous allons "broadcaster" notre contexte dans notre application en l'ajoutant dans le **`main`** :
```tsx {11,40,42} showLineNumbers
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';

import { RouterProvider, createBrowserRouter } from 'react-router-dom';
import App from './components/App/index.tsx';
import HomePage from './components/pages/HomePage.tsx';
import AddPizzaPage from './components/pages/AddPizzaPage.tsx';
import RegisterPage from './components/pages/RegisterPage.tsx';
import LoginPage from './components/pages/LoginPage.tsx';
import { UserContextProvider } from './contexts/UserContext.tsx';

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [
      {
        path: '',
        element: <HomePage />,
      },
      {
        path: 'add-pizza',
        element: <AddPizzaPage />,
      },
      {
        path: 'register',
        element: <RegisterPage />,
      },
      {
        path: 'login',
        element: <LoginPage />,
      },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <UserContextProvider>
      <RouterProvider router={router} />
    </UserContextProvider>
  </React.StrictMode>,
);
```

L'application actuelle utilise un `outletContext` pour passer les données de l'utilisateur authentifié entre les pages. Nous allons garder cet `outletContext` pour les pizzas et les boissons, mais nous allons utiliser le nouveau contexte pour l'utilisateur authentifié.

Nous allons donc supprimer tout ce qui est lié à la gestion de l'état de `authenticatedUser` sein de `App`. Voici la nouvelle version de `App` :

```tsx {1,11,44,114-122,132} showLineNumbers
import { useEffect, useState, useCallback, useContext } from 'react';
import { Outlet } from 'react-router-dom';
import './App.css';
import Footer from '../Footer';
import Header from '../Header';
import {
  Drink,
  NewPizza,
  Pizza,
  PizzeriaContext,
  UserContextType,
} from '../../types';
import NavBar from '../Navbar';

import { UserContext } from '../../contexts/UserContext';

const drinks: Drink[] = [
  {
    title: 'Coca-Cola',
    image:
      'https://media.istockphoto.com/id/1289738725/fr/photo/bouteille-en-plastique-de-coke-avec-la-conception-et-le-chapeau-rouges-d%C3%A9tiquette.jpg?s=1024x1024&w=is&k=20&c=HBWfROrGDTIgD6fuvTlUq6SrwWqIC35-gceDSJ8TTP8=',
    volume: 'Volume: 33cl',
    price: '2,50 €',
  },
  {
    title: 'Pepsi',
    image:
      'https://media.istockphoto.com/id/185268840/fr/photo/bouteille-de-cola-sur-un-fond-blanc.jpg?s=1024x1024&w=is&k=20&c=xdsxwb4bLjzuQbkT_XvVLyBZyW36GD97T1PCW0MZ4vg=',
    volume: 'Volume: 33cl',
    price: '2,50 €',
  },
  {
    title: 'Eau Minérale',
    image:
      'https://media.istockphoto.com/id/1397515626/fr/photo/verre-deau-gazeuse-%C3%A0-boire-isol%C3%A9.jpg?s=1024x1024&w=is&k=20&c=iEjq6OL86Li4eDG5YGO59d1O3Ga1iMVc_Kj5oeIfAqk=',
    volume: 'Volume: 50cl',
    price: '1,50 €',
  },
];

const App = () => {
  const [actionToBePerformed, setActionToBePerformed] = useState(false);
  const [pizzas, setPizzas] = useState<Pizza[]>([]);
  const { authenticatedUser } = useContext<UserContextType>(UserContext);

  const fetchPizzas = useCallback(async () => {
    try {
      const pizzas = await getAllPizzas();
      setPizzas(pizzas);
    } catch (err) {
      console.error('HomePage::error: ', err);
    }
  }, []);

  useEffect(() => {
    fetchPizzas();
  }, [fetchPizzas]);

  async function getAllPizzas() {
    try {
      const response = await fetch('/api/pizzas');

      if (!response.ok)
        throw new Error(
          `fetch error : ${response.status} : ${response.statusText}`,
        );

      const pizzas = await response.json();

      return pizzas;
    } catch (err) {
      console.error('getAllPizzas::error: ', err);
      throw err;
    }
  }

  const addPizza = async (newPizza: NewPizza) => {
    try {
      if (!authenticatedUser) {
        throw new Error('You must be authenticated to add a pizza');
      }
      const options = {
        method: 'POST',
        body: JSON.stringify(newPizza),
        headers: {
          'Content-Type': 'application/json',
          Authorization: authenticatedUser.token,
        },
      };

      const response = await fetch('/api/pizzas', options); // fetch retourne une "promise" => on attend la réponse

      if (!response.ok)
        throw new Error(
          `fetch error : ${response.status} : ${response.statusText}`,
        );

      const createdPizza = await response.json(); // json() retourne une "promise" => on attend les données

      setPizzas([...pizzas, createdPizza]);
    } catch (err) {
      console.error('AddPizza::error: ', err);
    }
  };

  const handleHeaderClick = () => {
    setActionToBePerformed(true);
  };

  const clearActionToBePerformed = () => {
    setActionToBePerformed(false);
  };

  const fullPizzaContext: PizzeriaContext = {
    addPizza,
    pizzas,
    setPizzas,
    actionToBePerformed,
    setActionToBePerformed,
    clearActionToBePerformed,
    drinks,
  };

  return (
    <div className="page">
      <Header
        title="We love Pizza"
        version={0 + 1}
        handleHeaderClick={handleHeaderClick}
      />
      <main>
        <NavBar />
        <Outlet context={fullPizzaContext} />
      </main>
      <Footer />
    </div>
  );
};

export default App;
```

De plus, la `Navbar` ne doit plus recevoir d'information d'`App`, mais directement du contexte. Nous allons donc modifier la `Navbar` pour qu'elle utilise le contexte :

```tsx {3-5,8-10,20} showLineNumbers
import { useNavigate } from 'react-router-dom';
import './Navbar.css';
import { useContext } from 'react';
import { UserContext } from '../../contexts/UserContext';
import { UserContextType } from '../../types';

const NavBar = () => {
  const { authenticatedUser, clearUser } =
    useContext<UserContextType>(UserContext);
  const navigate = useNavigate();

  if (authenticatedUser) {
    return (
      <nav>
        <button onClick={() => navigate('/')}>Home</button>
        <button onClick={() => navigate('/add-pizza')}>
          Ajouter une pizza
        </button>
        <button onClick={() => clearUser()}>Se déconnecter</button>
        <p>Hello dear {authenticatedUser.username}</p>
      </nav>
    );
  }

  return (
    <nav>
      <button onClick={() => navigate('/')}>Home</button>
      <button onClick={() => navigate('/register')}>
        Créer un utilisateur
      </button>
      <button onClick={() => navigate('/login')}>Se connecter</button>
    </nav>
  );
};

export default NavBar;
```

Pour montrer à quel point il est simple de consommer le contexte, nous avons ajouté un message de bienvenue à l'utilisateur connecté (`"Hello dear ..."`).

Nous allons maintenant mettre à jour `LoginPage` et `RegisterPage` pour que ces pages utilisent le contexte.

Voici `LoginPage` :

```tsx {3-5,8} showLineNumbers
import { useState, SyntheticEvent, useContext } from 'react';
import './index.css';
import { useNavigate } from 'react-router-dom';
import { UserContextType } from '../../types';
import { UserContext } from '../../contexts/UserContext';

const LoginPage = () => {
  const { loginUser }: UserContextType = useContext(UserContext);

  const navigate = useNavigate();
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e: SyntheticEvent) => {
    e.preventDefault();
    try {
      await loginUser({ username, password });
      navigate('/');
    } catch (err) {
      console.error('LoginPage::error: ', err);
    }
  };

  const handleUsernameInputChange = (e: SyntheticEvent) => {
    const input = e.target as HTMLInputElement;
    setUsername(input.value);
  };

  const handlePasswordChange = (e: SyntheticEvent) => {
    const input = e.target as HTMLInputElement;
    setPassword(input.value);
  };

  return (
    <div>
      <h1>Connectez un utilisateur</h1>
      <form onSubmit={handleSubmit}>
        <label htmlFor="username">Username</label>
        <input
          value={username}
          type="text"
          id="username"
          name="username"
          onChange={handleUsernameInputChange}
          required
        />
        <label htmlFor="password">Password</label>
        <input
          value={password}
          type="text"
          id="password"
          name="password"
          onChange={handlePasswordChange}
          required
        />
        <button type="submit">S'authentifier</button>
      </form>
    </div>
  );
};

export default LoginPage;
```

Voici `RegisterPage` :

```tsx {3-5,8} showLineNumbers
import { useState, SyntheticEvent, useContext } from 'react';
import './index.css';
import { useNavigate } from 'react-router-dom';
import { UserContextType } from '../../types';
import { UserContext } from '../../contexts/UserContext';

const RegisterPage = () => {
  const { registerUser }: UserContextType = useContext(UserContext);

  const navigate = useNavigate();
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e: SyntheticEvent) => {
    e.preventDefault();
    try {
      await registerUser({ username, password });
      navigate('/');
    } catch (err) {
      console.error('RegisterPage::error: ', err);
    }
  };

  const handleUsernameInputChange = (e: SyntheticEvent) => {
    const input = e.target as HTMLInputElement;
    setUsername(input.value);
  };

  const handlePasswordChange = (e: SyntheticEvent) => {
    const input = e.target as HTMLInputElement;
    setPassword(input.value);
  };

  return (
    <div>
      <h1>Ajoutez un utilisateur</h1>
      <form onSubmit={handleSubmit}>
        <label htmlFor="username">Username</label>
        <input
          value={username}
          type="text"
          id="username"
          name="username"
          onChange={handleUsernameInputChange}
          required
        />
        <label htmlFor="password">Password</label>
        <input
          value={password}
          type="text"
          id="password"
          name="password"
          onChange={handlePasswordChange}
          required
        />
        <button type="submit">Créer le compte</button>
      </form>
    </div>
  );
};

export default RegisterPage;
```

Nous avons maintenant une application qui utilise un contexte pour gérer l'utilisateur connecté. Nous avons également vu comment consommer ce contexte dans différents composants.

Si nécessaire, vous pouvez trouver le code associé à ce tutoriel ici : [context](https://github.com/e-vinci/cae-theory-demos/tree/main/context).

### Plus de documentation
N'hésitez pas à consulter la documentation officielle de React si vous souhaitez plus d'informations sur les contextes :
- Documentation générale de React sur les contextes : [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)
- Documentation plus spécifique sur le hook `useContext` : [useContext](https://react.dev/reference/react/useContext)

## Résumé

- Nous utilisons le mécanisme des contextes pour stocker un state qui doit être largement accédé.
- Grâce à ce mécanisme, nous bénéficions des avantages des mises à jour des props sans devoir faire de prop drilling.
- En plus du state en lui-même, il est nécessaire d'exposer les fonction pour le modifier.
- Un composant spécial, le ProviderWrapper, s'occupe de "broadcaster" son state au travers du contexte.


---
# 4. Tests Unitaires

## Mise en place des tests unitaires avec Vitest 

### a) Installation des dépendances
Nous allons réaliser les tests unitaires de nos applications React avec `Vitest`.

Pour ce tutoriel, veuillez créer un nouveau projet nommé `unit-tests` sur base d'un copier/coller du dossier `front` de votre tutoriel `context`. Si nécessaire, vous pouvez trouver le code associé au dossier `front` ici : [dossier front du tuto context](https://github.com/e-vinci/cae-theory-demos/tree/main/context/front).

Pour ajouter `Vitest` à votre projet, vous pouvez exécuter la commande suivante :

```bash
npm install -D vitest @testing-library/react jsdom @vitest/coverage-v8
```

`jsdom` est un environnement de test JavaScript qui simule un navigateur web. Il est utilisé par `Vitest` pour exécuter les tests unitaires.

`@testing-library/react` est une bibliothèque de test qui fournit des utilitaires pour tester les composants React.

`@vitest/coverage-v8` est un plugin de couverture de code pour `Vitest` qui génère des rapports de couverture de code.

Veuillez configurer votre application Vite pour exécuter les tests unitaires avec `Vitest`. Pour cela, vous devez modifier votre fichier `vite.config.ts` :

```ts {1-2,10-14} showLineNumbers
// eslint-disable-next-line spaced-comment
/// <reference types="vitest/config" />

import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react-swc';
import checker from 'vite-plugin-checker';

// https://vitejs.dev/config/
export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/setupTests.ts',
  },

  plugins: [
    react(),
    checker({
      typescript: true,
    }),
  ],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
});
```

Veuillez créer un fichier vide nommé `setupTests.ts` dans le répertoire `/src` de votre projet. Ce fichier pourrait être utilisé pour configurer l'environnement de test. Par exemple, vous pourriez y importer des dépendances nécessaires pour les tests unitaires afin de ne plus les importer dans chaque fichier de test.

### b) Exécution des tests

Pour écrire des tests unitaires avec `Vitest`, vous pouvez créer un fichier **`NomComposant.test.tsx`** à côté de votre composant ou fonction à tester.

Par défaut, `Vitest` recherche les fichiers contenant **`.test`** ou **`.spec`** dans le nom de fichier pour exécuter les tests.

Pour exécuter les tests à l'aide d'une commande, vous devez ajouter un script dans votre fichier `package.json` :

```json
{
  "scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```

Vous pouvez exécuter les tests en utilisant la commande suivante :

```bash
npm test
```

Pour générer un rapport de couverture de code, vous pouvez exécuter la commande suivante :

```bash
npm run coverage
```

Pour exécuter les tests directement au sein de VS Code, vous pouvez installer l'extension `Vitest` : [Vitest](https://marketplace.visualstudio.com/items?itemName=alexjoverm.vitest)

## Où enregistrer les spécifications de test ?

Il existe trois principales approches pour enregistrer les spécifications de test :
- **Approche intégrée** : Les spécifications de test sont enregistrées dans le même fichier que le composant ou la fonction à tester. Cela permet de garder les tests à proximité du code source et de les maintenir ensemble.
- **Approche séparée dans le répertoire du composant** : Les spécifications de test sont enregistrées dans des fichiers séparés avec l'extension `.test.tsx` ou `.spec.tsx`. Cela permet de garder les tests séparés du code source, mais proche.
- **Approche séparée dans un répertoire dédié** : Les spécifications de test sont enregistrées dans un répertoire dédié, par exemple `/src/unit-tests`. Cela permet de garder les tests complètement séparés du code source.

Chaque approche a ses avantages et ses inconvénients. Il est important de choisir celle qui convient le mieux à votre projet et à votre équipe.  Il est recommandé de garder les tests à proximité du code source pour faciliter la maintenance et la compréhension du code.

Pour le Projet CAE, nous vous demandons de suivre **l'approche séparée dans le répertoire du composant**.


Plus d'information sur `Vitest` : [Vitest](https://vitest.dev/)


## Comment tester un composant simple ?

Dans un premier temps, nous allons tester le composant `LoginPage` qui est un composant intelligent dont ses dépendances à la gestion de l'état sont injectées à l'aide d'un contexte.

Nous allons réaliser un premier simple test unitaire pour vérifier que le composant `LoginPage` affiche deux labels ("Username" et "Password").

Veuillez créer un nouveau script nommé `/src/components/pages/LoginPage.test.tsx` :

```tsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import { describe, test, expect } from 'vitest';
import LoginPage from '../../components/pages/LoginPage';

describe('LoginPage', () => {
  test('renders a form with username and password inputs', () => {
    render(
      <MemoryRouter>
        <LoginPage />
      </MemoryRouter>,
    );
    const usernameInput = screen.getByLabelText('Username');
    const passwordInput = screen.getByLabelText('Password');

    expect(usernameInput).toBeTruthy();
    expect(passwordInput).toBeTruthy();
  });
});
```

Voici une explication du code du test unitaire :

**`describe`** :
- permet de regrouper des tests liés entre eux. Ici, cela regroupe les tests pour le composant `LoginPage`.
- `'LoginPage'` est le nom du groupe de tests, ce qui aide à organiser et à identifier les tests dans les rapports de test.

**`test`** (ou **`it`**) :
- est une fonction qui définit un cas de test individuel.
- `'renders a form with username and password inputs'` : est une description du cas de test. Elle explique ce que le test vérifie.
- La fonction fléchée **`() => { ... }`** contient le code du test.
- Est un synonyme de **`it`**.


**`render`** :
- est une fonction qui rend un composant React dans un environnement de test.

**`<MemoryRouter>`** :
- est un composant de `react-router-dom` qui simule un environnement de routage en mémoire.
- Il est utilisé ici pour fournir le contexte de routage nécessaire au composant LoginPage.
  `<LoginPage />` est le composant à tester.

**`screen`** :
- est un objet qui représente le DOM rendu.

**`getByLabelText`** :
- est une méthode de `screen` qui recherche un élément dans le DOM par son texte de label associé. Ici, il recherche un élément avec le label `'Username'`.

**`expect`** :
- est une fonction qui crée une assertion.

**`toBeTruthy`** :
- est une méthode d'assertion qui vérifie que la valeur est "truthy" (c'est-à-dire qu'elle n'est pas null, undefined, false, 0, NaN, ou une chaîne vide).
- **`expect(passwordInput).toBeTruthy();`** : cette assertion vérifie que l'élément `passwordInput` existe dans le DOM.

Pour exécuter ce test unitaire, vous pouvez utiliser la commande suivante :

```bash
npm run test
```

Il est bien sûr possible de le faire dans VS Code avec l'extension `Vitest`, en cliquant sur le bouton `Run Test` à côté de la description du test.


## Comment mocker un contexte ?

Pour tester un composant qui dépend d'un contexte, vous pouvez utiliser la fonction `render` de `@testing-library/react` pour envelopper le composant dans un `Provider` qui fournit les valeurs du contexte.

Voici le code que nous souhaitons tester :
```tsx {11} showLineNumbers
const LoginPage = () => {
  const { loginUser }: UserContextType = useContext(UserContext);

  const navigate = useNavigate();
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e: SyntheticEvent) => {
    e.preventDefault();
    try {
      await loginUser({ username, password });
      navigate('/');
    } catch (err) {
      console.error('LoginPage::error: ', err);
    }
  };
//...
}
```

Nous souhaitons ici injecter un contexte `UserContext` qui contient une fonction `loginUser` et vérifier que le composant `LoginPage` appelle correctement cette fonction lors de la soumission du formulaire.

Voici un nouveau test unitaire à ajouter dans le fichier `/src/components/pages/LoginPage.test.tsx` :

```tsx {1,3,5,21-52} showLineNumbers
import { render, screen, fireEvent } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import { describe, test, expect, vi } from 'vitest';
import LoginPage from '../../components/pages/LoginPage';
import { UserContext } from '../../contexts/UserContext';

describe('LoginPage', () => {
  test('renders a form with username and password inputs', () => {
    render(
      <MemoryRouter>
        <LoginPage />
      </MemoryRouter>,
    );
    const usernameInput = screen.getByLabelText('Username');
    const passwordInput = screen.getByLabelText('Password');

    expect(usernameInput).toBeTruthy();
    expect(passwordInput).toBeTruthy();
  });

  test('calls loginUser when the form is submitted', async () => {
    const loginUserMock = vi.fn();
    const mockContextValue = {
      authenticatedUser: undefined,
      registerUser: vi.fn(),
      loginUser: loginUserMock,
      clearUser: vi.fn(),
    };

    render(
      <MemoryRouter>
        <UserContext.Provider value={mockContextValue}>
          <LoginPage />
        </UserContext.Provider>
      </MemoryRouter>,
    );

    const usernameInput = screen.getByLabelText('Username');
    const passwordInput = screen.getByLabelText('Password');
    const submitButton = screen.getByRole('button', {
      name: /s'authentifier/i,
    });

    fireEvent.change(usernameInput, { target: { value: 'testuser' } });
    fireEvent.change(passwordInput, { target: { value: 'password' } });
    fireEvent.click(submitButton);

    expect(loginUserMock).toHaveBeenCalledWith({
      username: 'testuser',
      password: 'password',
    });
  });
});
```

**`vi.fn()`** :
- est une fonction de `Vitest` qui crée un mock de fonction.
- Elle est utilisée ici pour créer un mock de la fonction `loginUser` du contexte `UserContext`.
- Un mock est une fonction simulée qui enregistre les appels et les arguments passés à la fonction simulée.

**`<UserContext.Provider>`** :
- est un composant de `React` qui injecte les valeurs du contexte aux composants enfants.

**`fireEvent.change`** :
- est une fonction de `@testing-library/react` qui simule un événement de changement sur un élément du DOM.
- Elle est utilisée ici pour simuler la saisie de valeurs dans les champs de formulaire.

**`fireEvent.click`** :
- est une fonction de `@testing-library/react` qui simule un événement de clic sur un élément du DOM.
- Elle est utilisée ici pour simuler le clic sur le bouton de soumission du formulaire.

**`expect(loginUserMock).toHaveBeenCalledWith({ ... })`** :
- est une assertion qui vérifie que la fonction `loginUserMock` a été appelée avec les arguments spécifiés.

Pour exécuter les deux tests unitaires existants, vous pouvez utiliser la commande suivante :

```bash
npm run test
```

## Comment mocker un useNavigate ?

Pour tester un composant qui utilise `useNavigate` de `react-router-dom`, vous pouvez utiliser la fonction `vi.mock` pour mocker le module `react-router-dom`.

Si l'on reprend le composant `LoginPage` que nous souhaitons tester :
```tsx {12} showLineNumbers
const LoginPage = () => {
  const { loginUser }: UserContextType = useContext(UserContext);

  const navigate = useNavigate();
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e: SyntheticEvent) => {
    e.preventDefault();
    try {
      await loginUser({ username, password });
      navigate('/');
    } catch (err) {
      console.error('LoginPage::error: ', err);
    }
  };
//...
}
```

Dans notre précédent test, nous avons déjà vérifié que la fonction `loginUser` est appelée correctement. Pour être complet dans notre deuxième test unitaire, nous allons vérifier que la fonction `navigate` est appelée avec le chemin '/' après l'appel à `loginUser`.

Voici le test unitaire à modifier dans le fichier `/src/components/pages/LoginPage.test.tsx` :

```tsx {1,2,7-13,29,31,39,64-66} showLineNumbers
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { MemoryRouter, useNavigate } from 'react-router-dom';
import { describe, test, expect, vi } from 'vitest';
import LoginPage from '../../components/pages/LoginPage';
import { UserContext } from '../../contexts/UserContext';

vi.mock('react-router-dom', async () => {
  const actual = await vi.importActual('react-router-dom');
  return {
    ...actual,
    useNavigate: vi.fn(),
  };
});

describe('LoginPage', () => {
  test('renders a form with username and password inputs', () => {
    render(
      <MemoryRouter>
        <LoginPage />
      </MemoryRouter>,
    );
    const usernameInput = screen.getByLabelText('Username');
    const passwordInput = screen.getByLabelText('Password');

    expect(usernameInput).toBeTruthy();
    expect(passwordInput).toBeTruthy();
  });

  test('calls loginUser and navigates to HomePage when the form is submitted', async () => {
    const loginUserMock = vi.fn();
    const navigateMock = vi.fn();
    const mockContextValue = {
      authenticatedUser: undefined,
      registerUser: vi.fn(),
      loginUser: loginUserMock,
      clearUser: vi.fn(),
    };

    vi.mocked(useNavigate).mockReturnValue(navigateMock);

    render(
      <MemoryRouter>
        <UserContext.Provider value={mockContextValue}>
          <LoginPage />
        </UserContext.Provider>
      </MemoryRouter>,
    );

    const usernameInput = screen.getByLabelText('Username');
    const passwordInput = screen.getByLabelText('Password');
    const submitButton = screen.getByRole('button', {
      name: /s'authentifier/i,
    });

    fireEvent.change(usernameInput, { target: { value: 'testuser' } });
    fireEvent.change(passwordInput, { target: { value: 'password' } });
    fireEvent.click(submitButton);

    expect(loginUserMock).toHaveBeenCalledWith({
      username: 'testuser',
      password: 'password',
    });

    await waitFor(() => {
      expect(navigateMock).toHaveBeenCalledWith('/');
    });
  });
});
  ```

Voici comment nous avons mocké `useNavigate` :
```tsx
vi.mock('react-router-dom', async () => {
  const actual = await vi.importActual('react-router-dom');
  return {
    ...actual,
    useNavigate: vi.fn(),
  };
});
```

**`vi.mock`** :
- est une fonction de `Vitest` qui permet de mocker un module.
- Elle est utilisée ici pour mocker le module `react-router-dom` et remplacer la fonction `useNavigate` par un mock.
- **`vi.importActual`** est une fonction qui permet d'importer le module réel pour le mocker.
- On récupère ici le module `react-router-dom` et on remplace uniquement la fonction `useNavigate` par un mock. On veut par exemple, garder la vrai fonction `MemoryRouter` pour ne pas la mocker.
- Si l'on avait souhaité mocker l'ensemble du module `react-router-dom`, on aurait pu simplement écrire **`vi.mock('react-router-dom', vi.fn())`**. Mais nous aurions eu le problème de ne pas pouvoir utiliser `MemoryRouter`...

Pour info, dans le composant `LoginPage`, voici comment nous utilisons `useNavigate` :
```tsx
const navigate = useNavigate();
```

**`useNavigate`** :
- est une fonction de `react-router-dom`, un hook, qui permet de naviguer vers une autre page.
- ce hook renvoie une fonction `navigate` qui peut être appelée pour naviguer vers une autre page.

Voici comment nous avons créé une nouvelle implémentations pour `useNavigate`, afin de renvoyer notre propre mock de la fonction `navigate` :
```tsx
const navigateMock = vi.fn();
//...
vi.mocked(useNavigate).mockReturnValue(navigateMock);
```

**`vi.mocked(useNavigate).mockReturnValue(navigateMock)`** :
- est une fonction de `Vitest` qui permet de mocker une méthode d'un module.
- Elle est utilisée ici pour mocker la méthode `useNavigate` du module `react-router-dom` et lui faire renvoyer notre mock `navigateMock`. C'est en fait l'équivalent de la fonction `navigate` que nous avons utilisée dans le composant `LoginPage`.

**`await waitFor(() => { ... })`** :
- est une fonction de `@testing-library/react` qui attend que la fonction de rappel passe un test.
- Elle est utilisée ici pour attendre que le mock `navigateMock` soit appelé avec le chemin '/'.

Veuillez exécuter vos deux tests unitaires.

## Comment lancer une exception à l'aide d'un mock ?

Si l'on reprend le composant `LoginPage` que nous souhaitons tester :
```tsx {11,13-15} showLineNumbers
const LoginPage = () => {
  const { loginUser }: UserContextType = useContext(UserContext);

  const navigate = useNavigate();
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e: SyntheticEvent) => {
    e.preventDefault();
    try {
      await loginUser({ username, password });
      navigate('/');
    } catch (err) {
      console.error('LoginPage::error: ', err);
    }
  };
//...
}
```

Nous souhaiterions tester le cas où la fonction `loginUser` lance une exception. Pour cela, nous allons mocker la fonction `loginUser` pour qu'elle lance une exception.

Nous allons aussi vérifier que la fonction `console.error` est appelée avec le message d'erreur. Pour ce faire, nous allons mocker `console.error` et vérifier que cette fonction est appelée avec le message d'erreur.

Voici le test unitaire à ajouter dans le fichier `/src/components/pages/LoginPage.test.tsx` :

```tsx {1,2,7-13,29,31,39,64-66} showLineNumbers
test('calls console.error when loginUser throws an error', async () => {
    const loginUserMock = vi
      .fn()
      .mockRejectedValueOnce(new Error('Login failed'));
    const consoleErrorMock = vi
      .spyOn(console, 'error')
      .mockImplementation(() => {});
    const mockContextValue = {
      authenticatedUser: undefined,
      registerUser: vi.fn(),
      loginUser: loginUserMock,
      clearUser: vi.fn(),
    };

    render(
      <MemoryRouter>
        <UserContext.Provider value={mockContextValue}>
          <LoginPage />
        </UserContext.Provider>
      </MemoryRouter>,
    );

    const usernameInput = screen.getByLabelText('Username');
    const passwordInput = screen.getByLabelText('Password');
    const submitButton = screen.getByRole('button', {
      name: /s'authentifier/i,
    });

    fireEvent.change(usernameInput, { target: { value: 'testuser' } });
    fireEvent.change(passwordInput, { target: { value: 'password' } });
    fireEvent.click(submitButton);

    expect(loginUserMock).toHaveBeenCalledWith({
      username: 'testuser',
      password: 'password',
    });

    await waitFor(() => {
      expect(consoleErrorMock).toHaveBeenCalledWith(
        'LoginPage::error: ',
        expect.any(Error),
      );
    });

    consoleErrorMock.mockRestore();
  });
```

**`vi.fn().mockRejectedValueOnce(new Error('Login failed'))`** :
- est une fonction de `Vitest` qui crée un mock de fonction qui lance une exception.
- Elle est utilisée ici pour créer un mock de la fonction `loginUser` qui lance une exception avec le message 'Login failed'.

**`vi.spyOn(console, 'error').mockImplementation(() => {})`** :
- est une fonction de `Vitest` qui crée un mock de la méthode `console.error`.
- Elle est utilisée ici pour créer un mock de la méthode `console.error` qui ne fait rien.

**`expect(consoleErrorMock).toHaveBeenCalledWith('LoginPage::error: ', expect.any(Error))`** :
- est une assertion qui vérifie que la méthode `console.error` a été appelée avec le message 'LoginPage::error: ' et une instance de la classe `Error`.

**`consoleErrorMock.mockRestore()`** :
- est une méthode qui restaure la méthode `console.error` à sa valeur d'origine.
- Elle est utilisée ici pour restaurer la méthode `console.error` après le test.

Veuillez exécuter vos trois tests unitaires.

Pour en savoir plus sur les tests, notamment pour éviter les redondances de code (`beforeAll`, `beforeEach`...), vous pouvez consulter la documentation de l'API de `Vitest` : [Vitest API](https://vitest.dev/api/)

Si nécessaire, vous pouvez trouver le code associé à ce tutorial ici : [unit-tests](https://github.com/e-vinci/cae-theory-demos/tree/main/unit-tests).

## Comment créer des rapports de tests ?

### Génération de rapport de tests

Pour générer des rapports de tests, vous pouvez exécuter la commande suivante :

```bash
npm run coverage
```

Cette commande génère un rapport de couverture de code dans le répertoire `/coverage`. Vous pouvez ouvrir le fichier `index.html` dans votre navigateur pour visualiser le rapport de couverture de code.

### Visualisation des tests via une UI

Pour visualiser les tests via une interface utilisateur, vous pouvez exécuter la commande suivante dans le répertoire de votre projet contenant des tests unitaires :

```bash
npx vitest --ui
```