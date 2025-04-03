# Deploying Prisma on Resource-Constrained Environments

## What Causes the "Out of Memory" Error (prisma:generate not working in the JS scripts section)

When deploying Node.js applications with Prisma on shared hosting or resource-constrained environments (like cPanel), you may encounter this error:

```
RangeError: WebAssembly.Instance(): Out of memory: Cannot allocate Wasm memory for new instance
```

![image_2025-04-03_03-15-00](https://github.com/user-attachments/assets/439dd8cb-aa36-4d0c-9ca1-e9e8192056e1)

![image_2025-04-03_03-15-22](https://github.com/user-attachments/assets/0a09f5dc-8db6-4c75-98b1-6adc656018b2)

![image_2025-04-03_03-16-00](https://github.com/user-attachments/assets/b47044c5-4e16-4d75-b458-c062160ac117)

![image_2025-04-03_03-16-28](https://github.com/user-attachments/assets/0141bfc1-7b4d-4f59-a278-286cd32f62ab)

This happens because Prisma uses WebAssembly (Wasm) modules during client generation that require more memory than many shared hosting environments allow. Prisma's client generation process is particularly memory-intensive, as it needs to parse your schema and generate type-safe database access code.

## Solution: Generate Locally and Deploy Pre-Generated Files

The solution is to generate the Prisma client on your local development machine (which has more resources) and then deploy the pre-generated files to your hosting environment.

## Step-by-Step Deployment Guide

### 1. Generate Prisma Client Locally

Run the Prisma generate command on your local development machine:

```bash
npx prisma generate
```

This will generate the Prisma client in `./node_modules/@prisma/client`
![image_2025-04-03_03-26-08](https://github.com/user-attachments/assets/541d5a92-a450-4d14-8bb0-5ed7efac7f56)

### 2. Compile Your TypeScript Project

Compile your project to JavaScript:

```bash
tsc
```

This creates the `dist` directory with your compiled JavaScript files.

### 3. Prepare the Prisma Client for Deployment

Zip the `.prisma` folder from your local `node_modules` directory:

```bash
cd node_modules
zip -r ../.prisma.zip .prisma
cd ..
```

Move the zipped folder to your project root.

### 4. Clean Up Unnecessary Files

Remove node_modules and package-lock.json (cPanel will reinstall these):

```bash
rm -rf node_modules package-lock.json
```

### 5. Package Your Project

Zip your entire project:

```bash
zip -r your-project.zip .
```

### 6. Upload and Extract on cPanel

Upload the zip file to your cPanel and extract it in your application directory.

### 7. Install Dependencies

In cPanel's Node.js App Manager:

- Navigate to your application
- Click "Run NPM Install" or run:

```bash
npm install
```

![image_2025-04-03_03-14-44](https://github.com/user-attachments/assets/3d6077c8-008f-481a-9eb1-2c32df5beea1)

This will regenerate node_modules directory but might fail when trying to generate the Prisma client.

### 8. Stop Your Application

Stop your application in the Node.js App Manager or via command line.
![image](https://github.com/user-attachments/assets/f9abd3e8-8959-4aa5-ab48-b95fcd9a0800)

### 9. Deploy Pre-Generated Prisma Client

Move the `.prisma.zip` file to the node_modules directory:

```bash
mv .prisma.zip node_modules/
cd node_modules
```

### 10. Remove Auto-Generated Prisma Files

If a `.prisma` directory was created during npm install, remove it:

```bash
rm -rf .prisma
```

![image_2025-04-03_03-18-02](https://github.com/user-attachments/assets/232748ab-9342-40a8-b0a5-8021185c6ec9)

### 11. Extract Your Pre-Generated Prisma Files

Extract the pre-generated files:

```bash
unzip .prisma.zip
```

This will create a `.prisma` folder with all the necessary files.

### 12. Restart Your Application

Restart your application in the Node.js App Manager or via command line.

### 13. Clean Up

Remove the zip file as it's no longer needed:

```bash
rm .prisma.zip
```

### 14. Verify Your Deployment

Test your application to ensure everything is working correctly!

## Notes for Future Updates

Remember that whenever you make changes to your Prisma schema, you'll need to:

1. Generate the client locally
2. Zip and deploy the `.prisma` directory again
3. Restart your application

## Troubleshooting

If you encounter issues:

- Check that the `.prisma` directory was correctly extracted in `node_modules`
- Ensure file permissions are correct (typically 755 for directories, 644 for files)
- Review server logs for any errors

## Yes, You Can Do It!

With this approach, you can successfully deploy Prisma applications even on resource-constrained environments!
