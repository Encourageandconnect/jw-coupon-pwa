export async function registerSW() {
  if ('serviceWorker' in navigator) {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js')
      return registration
    } catch (error) {
      console.error('Service worker registration failed:', error)
      return null
    }
  }
  return null
}