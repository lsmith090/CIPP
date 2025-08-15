# CIPP Styling Guide

This guide covers the styling approach used in CIPP, including Material-UI theming, custom components, responsive design patterns, and best practices for maintaining visual consistency.

## Theme Architecture

CIPP uses Material-UI v6+ with a custom theme system that supports light/dark modes and customizable color presets.

### Theme Structure

```
/src/theme/
├── index.js              # Main theme factory
├── colors.js             # Color definitions
├── utils.js              # Theme utilities
├── base/                 # Base theme configuration
│   ├── create-options.js # Breakpoints, shape, typography
│   ├── create-components.js # Component overrides
│   └── create-typography.js # Typography settings
├── light/                # Light mode theme
│   ├── create-options.js # Light mode factory
│   ├── create-palette.js # Light color palette
│   ├── create-shadows.js # Light shadows
│   └── create-components.js # Light component styles
└── dark/                 # Dark mode theme
    ├── create-options.js # Dark mode factory
    ├── create-palette.js # Dark color palette
    ├── create-shadows.js # Dark shadows
    └── create-components.js # Dark component styles
```

### Color System

CIPP uses a semantic color system with alpha variants for consistency:

```javascript
// /src/theme/colors.js
import { alpha } from "@mui/material/styles";

const withAlphas = (color) => ({
  ...color,
  alpha4: alpha(color.main, 0.04),   // Very light backgrounds
  alpha8: alpha(color.main, 0.08),   // Subtle highlights
  alpha12: alpha(color.main, 0.12),  // Light backgrounds
  alpha30: alpha(color.main, 0.3),   // Medium transparency
  alpha50: alpha(color.main, 0.5),   // High transparency
});

// Brand colors
export const blue = withAlphas({
  light: "#003049",
  main: "#003049",
  dark: "#003049",
  contrastText: "#FFFFFF",
});

export const orange = withAlphas({
  light: "#F77F00",
  main: "#F77F00", 
  dark: "#F77F00",
  contrastText: "#FFFFFF",
});
```

### Accessing Theme in Components

```javascript
import { useTheme } from '@mui/material/styles';
import { styled } from '@mui/material/styles';

// Hook approach
const MyComponent = () => {
  const theme = useTheme();
  
  return (
    <Box
      sx={{
        backgroundColor: theme.palette.primary.alpha12,
        color: theme.palette.text.primary,
        borderRadius: theme.shape.borderRadius,
        padding: theme.spacing(2)
      }}
    >
      Content
    </Box>
  );
};

// Styled component approach
const StyledCard = styled(Card)(({ theme }) => ({
  backgroundColor: theme.palette.background.paper,
  borderRadius: theme.shape.borderRadius,
  '&:hover': {
    backgroundColor: theme.palette.action.hover,
  }
}));
```

## Component Styling Patterns

### 1. SX Prop (Recommended)

Use the `sx` prop for most component styling:

```javascript
import { Box, Card, Typography } from '@mui/material';

const ComponentExample = () => (
  <Card
    sx={{
      p: 3,                             // padding: theme.spacing(3)
      m: 2,                             // margin: theme.spacing(2)
      borderRadius: 2,                  // borderRadius: theme.spacing(2)
      backgroundColor: 'background.paper',
      boxShadow: (theme) => theme.shadows[2],
      '&:hover': {
        boxShadow: (theme) => theme.shadows[4],
        transform: 'translateY(-2px)',
        transition: 'all 0.2s ease-in-out'
      },
      [theme => theme.breakpoints.down('md')]: {
        p: 2,                           // Responsive padding
        m: 1
      }
    }}
  >
    <Typography 
      variant="h6" 
      sx={{ 
        color: 'text.primary',
        fontWeight: 'bold',
        mb: 2 
      }}
    >
      Title
    </Typography>
  </Card>
);
```

### 2. Styled Components

For complex or reusable styles:

```javascript
import { styled } from '@mui/material/styles';
import { Card, alpha } from '@mui/material';

const GradientCard = styled(Card)(({ theme, variant = 'primary' }) => ({
  position: 'relative',
  overflow: 'hidden',
  background: variant === 'primary' 
    ? `linear-gradient(135deg, ${theme.palette.primary.main} 0%, ${theme.palette.primary.dark} 100%)`
    : `linear-gradient(135deg, ${theme.palette.secondary.main} 0%, ${theme.palette.secondary.dark} 100%)`,
  color: theme.palette.primary.contrastText,
  
  '&::before': {
    content: '""',
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    background: alpha(theme.palette.common.white, 0.1),
    transform: 'translateX(-100%)',
    transition: 'transform 0.3s ease',
  },
  
  '&:hover::before': {
    transform: 'translateX(0)',
  },
  
  [theme.breakpoints.down('md')]: {
    margin: theme.spacing(1),
  }
}));

// Usage
<GradientCard variant="primary">
  <CardContent>Content</CardContent>
</GradientCard>
```

### 3. Theme-aware Components

Create components that respond to theme changes:

```javascript
import { useTheme } from '@mui/material/styles';
import { useMediaQuery } from '@mui/material';

const ResponsiveComponent = ({ children }) => {
  const theme = useTheme();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));
  const isDark = theme.palette.mode === 'dark';
  
  return (
    <Box
      sx={{
        display: 'flex',
        flexDirection: isMobile ? 'column' : 'row',
        backgroundColor: isDark ? 'grey.900' : 'background.paper',
        border: `1px solid ${theme.palette.divider}`,
        borderRadius: theme.shape.borderRadius,
        p: theme.spacing(isMobile ? 2 : 3),
        gap: theme.spacing(2)
      }}
    >
      {children}
    </Box>
  );
};
```

## Layout and Spacing

### Spacing System

CIPP uses Material-UI's 8px-based spacing system:

```javascript
// Spacing values (multiply by 8px)
sx={{
  p: 1,    // padding: 8px
  m: 2,    // margin: 16px  
  px: 3,   // paddingLeft & paddingRight: 24px
  py: 4,   // paddingTop & paddingBottom: 32px
  mt: 5,   // marginTop: 40px
  gap: 2,  // gap: 16px
}}

// Custom spacing
sx={{
  padding: (theme) => theme.spacing(1.5), // 12px
  margin: (theme) => theme.spacing(0.5),  // 4px
}}
```

### Layout Patterns

#### Card Layouts

```javascript
// Standard card pattern
const InfoCard = ({ title, value, icon }) => (
  <Card
    sx={{
      p: 2,
      display: 'flex',
      alignItems: 'center',
      gap: 2,
      minHeight: 120,
      '&:hover': {
        boxShadow: (theme) => theme.shadows[4]
      }
    }}
  >
    <Avatar
      sx={{
        backgroundColor: 'primary.alpha12',
        color: 'primary.main',
        width: 56,
        height: 56
      }}
    >
      {icon}
    </Avatar>
    <Box>
      <Typography variant="overline" color="text.secondary">
        {title}
      </Typography>
      <Typography variant="h6">{value}</Typography>
    </Box>
  </Card>
);

// Full-width content card
const ContentCard = ({ children, actions }) => (
  <Card
    sx={{
      display: 'flex',
      flexDirection: 'column',
      height: '100%'
    }}
  >
    <CardContent sx={{ flexGrow: 1, p: 3 }}>
      {children}
    </CardContent>
    {actions && (
      <CardActions sx={{ px: 3, pb: 3, pt: 0 }}>
        {actions}
      </CardActions>
    )}
  </Card>
);
```

#### Grid Layouts

```javascript
import { Grid } from '@mui/material';

const DashboardLayout = ({ widgets }) => (
  <Grid container spacing={3}>
    {widgets.map((widget, index) => (
      <Grid 
        item 
        xs={12} 
        sm={6} 
        md={4} 
        key={index}
      >
        <InfoCard {...widget} />
      </Grid>
    ))}
  </Grid>
);

// Responsive grid with different breakpoints
const ResponsiveGrid = ({ items }) => (
  <Grid container spacing={2}>
    {items.map((item, index) => (
      <Grid
        item
        xs={12}      // Full width on mobile
        sm={6}       // Half width on small screens
        md={4}       // Third width on medium screens
        lg={3}       // Quarter width on large screens
        key={index}
      >
        <ItemCard {...item} />
      </Grid>
    ))}
  </Grid>
);
```

## Dark Mode Support

### Palette Configuration

CIPP automatically handles dark mode through the theme system:

```javascript
// Light mode colors are automatically adapted for dark mode
const LightModeComponent = () => (
  <Card
    sx={{
      backgroundColor: 'background.paper',  // Automatically adapts
      color: 'text.primary',               // Automatically adapts
      border: '1px solid',
      borderColor: 'divider'               // Automatically adapts
    }}
  >
    Content
  </Card>
);
```

### Manual Dark Mode Handling

When you need specific dark mode behavior:

```javascript
import { useTheme } from '@mui/material/styles';

const ThemeAwareComponent = () => {
  const theme = useTheme();
  const isDark = theme.palette.mode === 'dark';
  
  return (
    <Box
      sx={{
        backgroundColor: isDark 
          ? alpha(theme.palette.primary.main, 0.1)
          : alpha(theme.palette.primary.main, 0.05),
        backgroundImage: isDark
          ? 'none'
          : 'linear-gradient(45deg, transparent 30%, rgba(255,255,255,0.1) 50%)',
      }}
    >
      Content
    </Box>
  );
};
```

## Responsive Design

### Breakpoint System

CIPP uses custom breakpoints:

```javascript
// Breakpoint values (from /src/theme/base/create-options.js)
{
  xs: 0,     // Extra small devices
  sm: 600,   // Small devices
  md: 900,   // Medium devices  
  lg: 1200,  // Large devices
  xl: 1440,  // Extra large devices
}
```

### Responsive Styling

```javascript
import { useMediaQuery } from '@mui/material';

// Hook-based responsive design
const ResponsiveComponent = () => {
  const theme = useTheme();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));
  const isTablet = useMediaQuery(theme.breakpoints.between('md', 'lg'));
  
  return (
    <Typography 
      variant={isMobile ? 'h6' : 'h4'}
      sx={{
        textAlign: isMobile ? 'center' : 'left',
        fontSize: {
          xs: '1.2rem',
          sm: '1.5rem', 
          md: '2rem',
          lg: '2.5rem'
        }
      }}
    >
      Responsive Text
    </Typography>
  );
};

// Sx prop responsive styling
const ResponsiveCard = () => (
  <Card
    sx={{
      p: { xs: 2, sm: 3, md: 4 },           // Responsive padding
      flexDirection: { xs: 'column', md: 'row' }, // Layout change
      minHeight: { xs: 'auto', md: 200 },    // Conditional min height
      display: { xs: 'block', sm: 'flex' },  // Display change
    }}
  >
    Content
  </Card>
);
```

### Mobile-First Design

Always design mobile-first and enhance for larger screens:

```javascript
const MobileFirstCard = () => (
  <Card
    sx={{
      // Mobile styles (default)
      p: 2,
      flexDirection: 'column',
      textAlign: 'center',
      
      // Tablet and up
      [theme => theme.breakpoints.up('md')]: {
        p: 3,
        flexDirection: 'row',
        textAlign: 'left',
      },
      
      // Desktop and up
      [theme => theme.breakpoints.up('lg')]: {
        p: 4,
        minHeight: 200,
      }
    }}
  >
    Content
  </Card>
);
```

## Custom Scrollbars

CIPP includes custom scrollbar styling defined globally:

```javascript
// Global scrollbar styles (from /src/theme/index.js)
MuiCssBaseline: {
  styleOverrides: {
    html: {
      "--sb-track-color": "#232E33",
      "--sb-thumb-color": "#6BAF8D", 
      "--sb-size": "7px",
    },
    "html, body, *": {
      "&::-webkit-scrollbar": {
        width: "var(--sb-size)",
        height: "var(--sb-size)",
      },
      "&::-webkit-scrollbar-track": {
        backgroundColor: "var(--sb-track-color)",
        borderRadius: "4px",
      },
      "&::-webkit-scrollbar-thumb": {
        backgroundColor: "var(--sb-thumb-color)",
        borderRadius: "4px",
      }
    }
  }
}
```

## Animation and Transitions

### Standard Transitions

Use consistent transition timing:

```javascript
const AnimatedCard = () => (
  <Card
    sx={{
      transition: 'all 0.2s ease-in-out',
      transform: 'translateY(0)',
      '&:hover': {
        transform: 'translateY(-4px)',
        boxShadow: (theme) => theme.shadows[8],
      }
    }}
  >
    Content
  </Card>
);

// Loading animations
const LoadingBox = () => (
  <Box
    sx={{
      opacity: 0,
      animation: 'fadeIn 0.3s ease-in-out forwards',
      '@keyframes fadeIn': {
        from: { opacity: 0, transform: 'translateY(20px)' },
        to: { opacity: 1, transform: 'translateY(0)' }
      }
    }}
  >
    Content
  </Box>
);
```

### Skeleton Loading

Use Material-UI Skeleton for loading states:

```javascript
import { Skeleton } from '@mui/material';

const LoadingCard = () => (
  <Card sx={{ p: 2 }}>
    <Skeleton variant="circular" width={40} height={40} />
    <Skeleton variant="text" sx={{ mt: 1 }} />
    <Skeleton variant="text" width="60%" />
    <Skeleton variant="rectangular" height={100} sx={{ mt: 2 }} />
  </Card>
);
```

## Typography

### Typography Scale

CIPP uses Material-UI's typography system with custom configurations:

```javascript
// Typography variants
<Typography variant="h1">Main Heading</Typography>      // Largest
<Typography variant="h2">Section Heading</Typography>   
<Typography variant="h3">Subsection Heading</Typography>
<Typography variant="h4">Page Title</Typography>        // Common page title
<Typography variant="h5">Card Title</Typography>
<Typography variant="h6">Small Heading</Typography>     // Common card title
<Typography variant="body1">Body Text</Typography>      // Default body
<Typography variant="body2">Secondary Text</Typography>
<Typography variant="caption">Caption Text</Typography>
<Typography variant="overline">Label Text</Typography>  // Common for labels
```

### Custom Typography

```javascript
const CustomText = () => (
  <Typography
    sx={{
      fontSize: { xs: '1rem', md: '1.2rem' },
      fontWeight: 'bold',
      color: 'primary.main',
      textTransform: 'uppercase',
      letterSpacing: '0.05em',
      lineHeight: 1.6
    }}
  >
    Custom Styled Text
  </Typography>
);
```

## Best Practices

### 1. Consistency
- Use theme tokens instead of hard-coded values
- Follow established color and spacing patterns
- Maintain consistent component sizing and spacing

### 2. Performance
- Prefer `sx` prop over styled components for simple styles
- Use styled components for complex, reusable styles
- Avoid inline styles that create new objects on each render

### 3. Accessibility
- Ensure sufficient color contrast (4.5:1 for normal text)
- Use semantic color meanings (error for errors, success for success)
- Test in both light and dark modes

### 4. Responsiveness
- Design mobile-first
- Use breakpoint-based responsive design
- Test on various screen sizes

### 5. Theme Integration
- Always use theme values for colors, spacing, and typography
- Support both light and dark modes
- Use theme hooks and utilities appropriately

## Common Anti-Patterns

1. **Hard-coded colors**: Use theme palette instead
```javascript
// Bad
sx={{ color: '#FF0000' }}

// Good  
sx={{ color: 'error.main' }}
```

2. **Pixel-perfect spacing**: Use theme spacing
```javascript
// Bad
sx={{ margin: '16px' }}

// Good
sx={{ m: 2 }}
```

3. **Inconsistent breakpoints**: Use theme breakpoints
```javascript
// Bad
sx={{ [mediaQuery]: { display: 'none' } }}

// Good
sx={{ display: { xs: 'none', md: 'block' } }}
```

4. **Ignoring dark mode**: Always consider both themes
```javascript
// Bad (only works in light mode)
sx={{ backgroundColor: '#FFFFFF' }}

// Good (works in both modes)
sx={{ backgroundColor: 'background.paper' }}
```

5. **Creating objects in render**: Memoize complex styles
```javascript
// Bad
const styles = { complexObject: 'values' };

// Good
const styles = useMemo(() => ({ complexObject: 'values' }), [dependencies]);
```

This styling guide ensures consistent, maintainable, and accessible styling across the CIPP application. Always refer to existing components for patterns and maintain consistency with the established design system.