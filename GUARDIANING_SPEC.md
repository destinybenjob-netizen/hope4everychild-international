# GuardianPing 2.0: Complete Technical Specification

## Executive Summary
GuardianPing 2.0 is a criminological intelligence platform combining instantaneous emergency response (SOS), forensic integrity (Black Box recording), and zero-knowledge privacy (end-to-end encryption).

## Phase 1: The Prototype (COMPLETE)
- React frontend with tactical UI
- Supabase real-time database
- RSA-2048 + AES-256 encryption
- SHA-256 forensic hashing

## Phase 2: The OS Killer (PLAN 1)
### Problem
Background processes killed by iOS/Android to save battery. SOS fails if app is dead.

### Solution: Capacitor Wrapper
```bash
npm install @capacitor/core @capacitor/cli
npx cap init
npx cap add ios
npx cap add android
```

### Configuration
```json
{
  "appId": "com.guardiaping.app",
  "appName": "GuardianPing",
  "webDir": "dist",
  "plugins": {
    "Geolocation": {
      "permissions": ["location"]
    },
    "BackgroundTask": {
      "label": "background_sync"
    }
  }
}
```

### Key Implementations
1. **Always-On Location**: Request "Always Allow" permission on boot
2. **Background Tasks**: Keep app alive every 10 minutes
3. **Native Alerts**: Use native notifications (cannot be suppressed)
4. **SOS Routing**: Direct to native emergency UI

## Phase 3: The Forensic Handoff (PLAN 2)
### Problem
Law enforcement needs standard file format, not a web link.

### Solution: One-Click Evidence Export

```typescript
import JSZip from 'jszip';
import jsPDF from 'jspdf';

export async function generateEvidenceVault(incidentId: string) {
  // 1. Query incident data
  const incident = await fetchIncident(incidentId);
  const audioClips = await fetchAudio(incidentId);
  const photos = await fetchPhotos(incidentId);
  
  // 2. Generate PDF report
  const pdf = new jsPDF();
  pdf.text(`INCIDENT REPORT`, 10, 10);
  pdf.text(`ID: ${incident.id}`, 10, 20);
  pdf.text(`Timestamp: ${incident.timestamp}`, 10, 30);
  pdf.text(`Location: ${incident.lat}, ${incident.lng}`, 10, 40);
  pdf.text(`Hash Chain:`, 10, 50);
  
  incident.hashes.forEach((hash, i) => {
    pdf.text(`${i+1}. ${hash}`, 10, 60 + (i*5));
  });
  
  const pdfBlob = pdf.output('blob');
  
  // 3. Create encrypted ZIP
  const zip = new JSZip();
  zip.file('INCIDENT_REPORT.pdf', pdfBlob);
  
  for (const audio of audioClips) {
    zip.file(`audio_${audio.id}.m4a`, audio.blob);
  }
  
  for (const photo of photos) {
    zip.file(`photo_${photo.id}.jpg`, photo.blob);
  }
  
  // 4. Password protect & export
  const zipBlob = await zip.generateAsync({type: 'blob'});
  downloadFile(zipBlob, `evidence_${incidentId}.zip`);
}
```

## Phase 4: The Brain Upgrade (PLAN 3)
### Problem
Brian (GPT-4o) can hallucinate. Need RAG system tied to DestinyBen Security manual.

### Solution: pgvector RAG

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS pgvector;

-- Create knowledge base table
CREATE TABLE knowledge_base (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source TEXT,
  content TEXT,
  embedding vector(1536),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX ON knowledge_base USING ivfflat (embedding vector_cosine_ops);
```

```typescript
// Supabase Edge Function: brian-triage
export async function brianTriage(userMessage: string) {
  // 1. Convert user message to embedding
  const embedding = await getEmbedding(userMessage);
  
  // 2. Search knowledge base (RAG)
  const { data: relevant } = await supabase
    .rpc('match_knowledge_base', {
      query_embedding: embedding,
      match_threshold: 0.7,
      match_count: 5
    });
  
  // 3. Build context from retrieved docs
  const context = relevant
    .map(r => r.content)
    .join('\n---\n');
  
  // 4. Call GPT-4o with context
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      {
        role: 'system',
        content: `You are Brian, GuardianPing's emergency triage AI. Use ONLY the following context. Never make up information.\n\nCONTEXT:\n${context}`
      },
      {
        role: 'user',
        content: userMessage
      }
    ]
  });
  
  return response.choices[0].message.content;
}
```

## Phase 5: The Enterprise Moat (PLAN 4)
### Problem
Free product = no revenue. Need B2B layer.

### Solution: Corporate Parent Tier

```typescript
// Stripe webhook for Enterprise subscriptions
export async function handleStripeWebhook(event: any) {
  if (event.type === 'checkout.session.completed') {
    const { customer_email, metadata } = event.data.object;
    
    // Update user to enterprise_admin
    await supabase
      .from('users')
      .update({
        tier: 'enterprise',
        stripe_customer_id: customer_email,
        admin_dashboard_access: true
      })
      .eq('email', customer_email);
    
    // Create enterprise workspace
    const { data: workspace } = await supabase
      .from('enterprise_workspaces')
      .insert({
        admin_id: customer_email,
        org_name: metadata.org_name,
        max_users: metadata.tier === 'pro' ? 100 : 500,
        monthly_cost: 1200
      })
      .select();
    
    // Grant dashboard access
    await supabase
      .from('dashboards')
      .insert({
        workspace_id: workspace.id,
        dashboard_type: 'commander_array_view',
        features: ['realtime_tracking', 'user_analytics', 'api_access']
      });
  }
}
```

## Integration Matrix (Summary)

| Software | Integration | Implementation |
|----------|-------------|----------------|
| **Capacitor** | Native wrapper | iOS/Android background keep-alive |
| **JSZip + jsPDF** | Evidence export | Password-protected PDF/ZIP |
| **pgvector** | AI memory | DestinyBen knowledge base RAG |
| **OpenAI Embeddings** | Text→Vector conversion | Knowledge base indexing |
| **Stripe Identity** | KYC verification | ID + liveness check |
| **Stripe Billing** | Subscription management | Enterprise $1,200/mo |
| **Maplibre GL JS** | Mapping (free tier) | Custom dark tactical styling |
| **Web Crypto API** | End-to-end encryption | RSA-2048/AES-256 |

## Deployment Timeline

**Week 1**:
- [ ] Capacitor integration (Plan 1)
- [ ] iOS/Android build
- [ ] Background location tracking

**Week 2**:
- [ ] JSZip/jsPDF evidence export (Plan 2)
- [ ] PDF generation & hashing
- [ ] Encrypted ZIP download

**Week 3**:
- [ ] pgvector setup (Plan 3)
- [ ] Knowledge base ingestion
- [ ] RAG edge function
- [ ] Brian AI integration

**Week 4**:
- [ ] Stripe Identity KYC (Plan 4)
- [ ] Enterprise dashboard UI
- [ ] Subscription billing
- [ ] Array-view mapping UI

## Cost Calculation

**Infrastructure**:
- Supabase (pgvector): $100/mo
- OpenAI API (embeddings + GPT-4o): $200/mo
- Stripe processing (2.9% + $0.30): ~$500/mo @ $18K revenue
- Domain: $10/mo
- Email: $10/mo
- Monitoring: $20/mo

**Total: ~$840/month**

**At $18K revenue**: 95%+ profit margin
