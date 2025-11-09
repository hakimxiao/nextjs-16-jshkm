UPDATE DI NEXT 16  :
    react compiler support sekarang stable. Ini secara otomatis melakukkan memoise component untuk
    komponen yang tidak perlu render ulang. Sehingga kita tidak perlu lagi menggunakan useMemo dan useCallback hooks
    karena compiler secara otomatis melakukkan analisa kode kemudian mengoptinmalkan prilaku rendering secara otomatis.
    
-   Untuk menghadirklan fitur ini kita harus menginstalll plugin via terminal :
  - npm install babel-plugin-react-compiler@latest
    - kemudian kita pergi ke next config untuk melakukkan set reactCompiler ke True
      - ini hasilnya :
        - import type { NextConfig } from "next";
              const nextConfig: NextConfig = {
              reactCompiler: true,
              experimental: {
              turbopackFileSystemCacheForDev: true,
            }
          };
        
          export default nextConfig;
